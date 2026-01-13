# Estate Rental Project - Kubernetes Deployment

Ce projet a été restructuré pour supporter un déploiement hybride (Local sur Minikube et Cloud sur AWS EKS) en utilisant **Kustomize**.

## Architecture des Dossiers

- **k8s/base/** : Contient les manifestes communs (Deployments, Services, ConfigMaps, Secrets).
    - Les Services sont de type `ClusterIP`.
    - Les configurations sont externalisées dans `configmap.yaml` et `secret.yaml`.
- **k8s/overlays/minikube/** : Configuration pour le développement local.
    - Patch les services Gateway et Frontend en `NodePort`.
    - Inclut l'infrastructure locale (MySQL, Kafka, Zookeeper).
- **k8s/overlays/eks/** : Configuration pour AWS EKS.
    - Prépare l'Ingress (ALB Controller).
    - Configure les endpoints RDS via des patches.
- **k8s/monitoring/** : Configuration Prometheus/Grafana (ServiceMonitor).

## Prérequis

- Kubernetes Cluster (Minikube ou EKS)
- `kubectl` installé
- `kustomize` (intégré à kubectl depuis v1.14)

## Déploiement

### 1. Local (Minikube)

Pour déployer l'application complète avec l'infrastructure locale :

```bash
kubectl apply -k k8s/overlays/minikube
```

**Accès :**
- Frontend : `http://<minikube-ip>:30420`
- Gateway : `http://<minikube-ip>:30080`

### 2. AWS (EKS)

Pour déployer sur EKS (nécessite un cluster EKS avec ALB Controller configuré) :

1. Mettre à jour `k8s/overlays/eks/patch-db-endpoint.yaml` avec vos endpoints RDS réels.
2. Appliquer la configuration :

```bash
kubectl apply -k k8s/overlays/eks
```

## Gestion des Secrets

> [!WARNING]
> Les secrets (mots de passe DB) sont actuellement stockés encodés en base64 dans `k8s/base/secret.yaml`.
> **POUR LA PRODUCTION** : Il est impératif d'utiliser une solution sécurisée comme AWS Secrets Manager, HashiCorp Vault ou SealedSecrets, et de ne pas commiter ce fichier dans le dépôt public.

## Monitoring

Pour activer le monitoring (nécessite Prometheus Operator) :

```bash
kubectl apply -k k8s/monitoring
```

## Roadmap AWS EKS

1. **Base de Données** : Migrer MySQL local vers Amazon RDS. Mettre à jour les endpoints dans `k8s/overlays/eks/patch-db-endpoint.yaml`.
2. **Ingress** : Configurer AWS Load Balancer Controller pour gérer l'Ingress défini dans `k8s/overlays/eks/ingress.yaml`.
3. **Sécurité** : Remplacer `base/secret.yaml` par ExternalSecrets Operator couplé à AWS Secrets Manager.
4. **Scaling** : Ajuster les `resources.limits` et configurer Horizontal Pod Autoscaler (HPA) pour les microservices critiques.
