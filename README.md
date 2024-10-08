# Argo CD App of Apps Repository

This repository follows the Argo CD App of Apps pattern to manage deployment of multiple applications across different environments using Helm charts.

## Repository Structure
```
my-argocd-project/
├── root-app.yaml
├── README.md
├── apps/
│ ├── app1.yaml
│ ├── app2.yaml
│ │ ├── app1.yaml
| |
├── values/
| ├──app-1
|   |── app1-dev-values.yaml
|   |── app1-staging-values.yaml
|   |── app1-prod-values.yaml
| |
| ├──app-2
|   |── app2-dev-values.yaml
|   |── app2-staging-values.yaml
|   |── app2-prod-values.yaml
```

### Explanation

- **README.md:** General documentation about the repository structure and guidelines.
- **environments/:** Contains directories for each environment (`dev`, `staging`, `production`).
- **app1.yaml, app2.yaml, app3.yaml:** Argo CD Application resource definitions for managing the Helm releases.
- **values.yaml:** Values file for the Helm chart specific to this application and environment.
- **root-app.yaml:** The root Argo CD Application manifest following the App of Apps pattern.

### Example Root Application Manifest (`root-app.yaml`)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: root-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/your-repo.git'
    targetRevision: HEAD
    path: environments/dev
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: argocd
  syncPolicy:
    automated:
      selfHeal: true
```

### Example Child Application Manifest (`/apps/app1.yaml`)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: app1
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/your-repo.git'
    targetRevision: HEAD
    path: environments/dev/app1
    helm:
      valueFiles:
        - values.yaml
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: app1-dev
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
```

### Example Values File (`/values/myapp/values.yaml`)

```yaml
replicaCount: 1
image:
  repository: myapp/app1
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi

nodeSelector: {}

tolerations: []

affinity: {}
```