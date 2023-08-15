### Using Overlays

#### Here's how you can use overlays in your GitOps setup for a multi-cluster application deployment with an active-passive BCP, using Flux:

```
my-gitops-repo/
├── base/
│   └── nginx-deployment.yaml
├── overlays/
│   ├── production/
│   │   ├── primary/
│   │   │   └── kustomization.yaml
│   │   └── secondary/
│   │       └── kustomization.yaml
└── ...
```
##### Base Manifests:
Create a base Kubernetes manifest for the Nginx deployment in the base directory:

base/nginx-deployment.yaml:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 0   # Set replicas to 0 as the default
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
```
Overlays:
Define overlays for both the primary and secondary clusters in the overlays directory:

overlays/production/primary/kustomization.yaml:
```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ../../base
patchesStrategicMerge:
  - nginx-deployment-primary.yaml
```
overlays/production/primary/nginx-deployment-primary.yaml:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3   # Set replicas to 3 for the primary cluster
```

overlays/production/secondary/kustomization.yaml:
```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ../../base
patchesStrategicMerge:
  - nginx-deployment-secondary.yaml
```
overlays/production/secondary/nginx-deployment-secondary.yaml:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 0   # Set replicas to 0 for the secondary cluster
```
##### Failover Automation:
Create automation scripts to trigger a Flux sync for the secondary cluster and update the appropriate overlay file for scaling replicas in case of a failure.

Using overlays simplifies the management of different environment configurations, and it allows you to keep your base manifests clean and unchanged. This approach also makes it easier to maintain consistency across different environments while providing the flexibility to adapt to specific needs.

### Pros
* Can use a single repo to deploy in different/several clusters
* Central deployment definition under base folder which is referenced by primary and secondary  cluster folders
* Suitable for small teams with fewer deployments


### Cons
- Relies on external scripts/manual modifications during switch-over
- requires deployment abstraction like IDPs in the case of an enterprise setup
