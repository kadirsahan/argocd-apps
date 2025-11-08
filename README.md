# ArgoCD Apps - GitOps Repository

Central GitOps repository for managing ArgoCD applications using the App-of-Apps pattern.

## Overview

This repository contains ArgoCD Application manifests that define what applications should be deployed to the Kubernetes cluster. It follows the **App-of-Apps pattern**, where a root application deploys and manages all child applications.

## Architecture

```
argocd-apps (this repo)
├── app-of-apps.yaml          # Root application
└── apps/
    ├── kraft-kafka-server.yaml   # Kafka with KRaft mode
    └── kafka-connect.yaml        # Kafka Connect
```

## Applications

### 1. kraft-kafka-server
- **Repository**: https://github.com/kadirsahan/kraft-kafka-server
- **Namespace**: kraft-kafka-server
- **Sync Wave**: 1 (deployed first)
- **Description**: Apache Kafka cluster running in KRaft mode (no ZooKeeper)

### 2. kafka-connect
- **Repository**: https://github.com/kadirsahan/kafka-connect
- **Namespace**: kafka-connect
- **Sync Wave**: 2 (deployed after Kafka)
- **Description**: Kafka Connect for data integration

## Prerequisites

- Kubernetes cluster (local or cloud)
- ArgoCD installed and running
- Access to the ArgoCD UI or CLI

## Deployment

### Step 1: Deploy the App-of-Apps

Once ArgoCD is installed, deploy the root application:

```bash
kubectl apply -f app-of-apps.yaml
```

### Step 2: Verify Deployment

Check the ArgoCD UI or use the CLI:

```bash
# List all applications
kubectl get applications -n argocd

# Check app-of-apps status
kubectl get application app-of-apps -n argocd

# Check individual applications
kubectl get application kraft-kafka-server -n argocd
kubectl get application kafka-connect -n argocd
```

### Step 3: Access ArgoCD UI

Open the ArgoCD UI to monitor the deployment:

```
http://localhost:30080
```

## Sync Waves

Applications are deployed in order using sync waves:

1. **Wave 1**: kraft-kafka-server (Kafka must be ready first)
2. **Wave 2**: kafka-connect (depends on Kafka)

## Sync Policy

All applications use automated sync with:
- **prune**: true - Remove resources not defined in Git
- **selfHeal**: true - Auto-sync when cluster state differs from Git
- **CreateNamespace**: true - Auto-create namespaces

## Adding New Applications

To add a new application:

1. Create a new YAML file in `apps/` directory:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-new-app
  namespace: argocd
  annotations:
    argocd.argoproj.io/sync-wave: "3"
spec:
  project: default
  source:
    repoURL: https://github.com/kadirsahan/my-new-app.git
    targetRevision: main
    path: .
  destination:
    server: https://kubernetes.default.svc
    namespace: my-new-app
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

2. Commit and push to Git
3. ArgoCD will automatically detect and deploy the new application

## Removing Applications

To remove an application:

1. Delete the YAML file from `apps/` directory
2. Commit and push to Git
3. ArgoCD will automatically remove the application (due to prune: true)

## Troubleshooting

### Application Not Syncing

```bash
# Check application status
kubectl describe application <app-name> -n argocd

# Force sync via CLI
argocd app sync <app-name>

# Check application logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller
```

### Application Stuck in Progressing

```bash
# Check the resources
kubectl get all -n <namespace>

# Check events
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
```

## Related Repositories

- [argocd-gitops](https://github.com/kadirsahan/argocd-gitops) - ArgoCD installation
- [kraft-kafka-server](https://github.com/kadirsahan/kraft-kafka-server) - Kafka cluster
- [kafka-connect](https://github.com/kadirsahan/kafka-connect) - Kafka Connect

## Directory Structure

```
argocd-apps/
├── app-of-apps.yaml          # Root application manifest
├── apps/                      # Individual application manifests
│   ├── kraft-kafka-server.yaml
│   └── kafka-connect.yaml
└── README.md                  # This file
```

## License

MIT

## Author

kadirsahan
