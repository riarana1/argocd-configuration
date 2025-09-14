## Kubernetes GitOps Configuration Example with Argo CD

Here's the current structure of the repo (will be modified soon):

```shell
├── apps
│   └── cert-manager
│       ├── secrets.yaml
│       ├── certificates.yaml
│       └── cluster-issuer.yaml
└── bootstrap
    ├── Chart.yaml
    ├── templates
    │   ├── cert-manager.yaml
    │   ├── postgresql.yaml
    │   ├── projects.yaml
    │   ├── vault.yaml
    │   └── vault-config-operator.yaml
    └── values.yaml
```

For now, it shows how to install and use in the GitOps way:

1. [cert-manager](https://cert-manager.io/) via Helm chart

For more information you can refer to the articles:

1. [Testing GitOps on Virtual Kubernetes Clusters with ArgoCD](https://github.com/riarana1/argocd-config/)

## Getting Started

You can test the configuration on the virtual Kubernetes cluster using the [vcluster](https://www.vcluster.com/) tool. In order to that create the ArgoCD `ApplicationSet` based on the cluster generator:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: config
  namespace: argocd
spec:
  generators:
    - clusters: # (1)
        selector:
          matchLabels:
            argocd.argoproj.io/secret-type: cluster
  template:
    metadata:
      name: '{{name}}-config-test'
    spec:
      destination:
        namespace: argocd
        server: https://kubernetes.default.svc
      project: default
      source:
        helm:
          parameters:
            - name: server
              value: '{{server}}'
            - name: project
              value: '{{metadata.labels.loft.sh/vcluster-instance-name}}'
        path: bootstrap
        repoURL: https://github.com/riarana1/argo-cd-config.git
        targetRevision: '{{metadata.labels.loft.sh/vcluster-instance-name}}'
      syncPolicy:
        automated: {}
        syncOptions:
          - CreateNamespace=true
```

If you just want to test on the single cluster managed by ArgoCD you can create the following ArgoCD `Application`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: config
  namespace: argocd
spec:
  destination:
    namespace: argocd
    server: https://kubernetes.default.svc
  project: default
  source:
    path: bootstrap
    repoURL: https://github.com/riarana1/argo-cd-config.git
    targetRevision: HEAD
  syncPolicy:
    automated: {}
    syncOptions:
      - CreateNamespace=true
```
