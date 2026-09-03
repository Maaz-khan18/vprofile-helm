# VProfile Kubernetes Deployment

Kubernetes deployment configuration for the VProfile application. The repository provides the application stack as a Helm chart and includes Argo CD resources for GitOps deployment.

## What is deployed

The Helm chart in `helm/vprofile` deploys:

- VProfile application frontend
- MySQL database with a persistent volume claim
- Memcached
- RabbitMQ
- Kubernetes services and application credentials
- Optional AWS Application Load Balancer ingress

The chart's default ingress host is `vprofile.maazkhan.xyz`.

## Repository layout

```text
.
├── argocd/
│   ├── apps/vprofile-app.yaml       # Argo CD Application
│   └── projects/vprofile-project.yaml
├── helm/vprofile/
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/                   # Helm resource templates
└── kubedefs/                        # Static Kubernetes manifests
```

The Helm chart is the recommended deployment method. The manifests in `kubedefs/` are an alternative, non-Helm representation of the stack.

## Prerequisites

- Kubernetes cluster and configured `kubectl` context
- Helm 3
- A Kubernetes storage class named `gp2`, or a custom storage class configured in `values.yaml`
- An ingress controller when external access is required
- AWS Load Balancer Controller and an ACM certificate when using the default ALB ingress
- Argo CD when using the GitOps workflow

## Deploy with Helm

Validate the chart before installation:

```bash
helm lint ./helm/vprofile
helm template vprofile ./helm/vprofile
```

Install the chart into the `vprofile` namespace:

```bash
helm upgrade --install vprofile ./helm/vprofile \
  --namespace vprofile \
  --create-namespace
```

For a production deployment, override the demo credentials and environment-specific values. For example:

```bash
helm upgrade --install vprofile ./helm/vprofile \
  --namespace vprofile \
  --create-namespace \
  --set secrets.dbPassword='<database-password>' \
  --set secrets.rmqPassword='<rabbitmq-password>' \
  --set ingress.host='<application-domain>'
```

Do not commit real passwords to `values.yaml`.

## Deploy with Argo CD

The Argo CD application is configured to:

- Read the chart from `helm/vprofile` on the `main` branch
- Deploy to the in-cluster Kubernetes API server
- Use the `vprofile` namespace
- Create the namespace automatically
- Automatically sync, prune removed resources, and self-heal drift

Apply the project and application definitions after Argo CD is installed:

```bash
kubectl apply -f argocd/projects/vprofile-project.yaml
kubectl apply -f argocd/apps/vprofile-app.yaml
```

Check synchronization status:

```bash
kubectl get application vprofile -n argocd
argocd app get vprofile
```

The Argo CD repository access must be configured for `git@github.com:Maaz-khan18/vprofile-helm.git` before synchronization can succeed.

## Configuration

Edit `helm/vprofile/values.yaml` or provide a separate values file. Important settings include:

| Setting | Default | Purpose |
| --- | --- | --- |
| `app.image` / `app.tag` | ECR image / `5705bed` | VProfile application image |
| `app.replicas` | `1` | Application replica count |
| `db.storageClass` | `gp2` | Storage class for MySQL |
| `db.storageSize` | `3Gi` | MySQL persistent storage size |
| `ingress.enabled` | `true` | Create the ingress resource |
| `ingress.host` | `vprofile.maazkhan.xyz` | Public application hostname |
| `dockerregistry.enabled` | `false` | Create and use a registry pull secret |

The default ingress template contains an AWS ACM certificate ARN. Update the annotation in `helm/vprofile/templates/ingress.yaml` for another AWS account, region, or certificate.

## Verify the deployment

```bash
kubectl get pods -n vprofile
kubectl get deploy,svc,pvc,ingress -n vprofile
helm status vprofile -n vprofile
```

Wait for all pods to become ready. The application deployment uses init containers to wait for the database and Memcached services. If the ingress is enabled, obtain its address with:

```bash
kubectl get ingress vpro-ingress -n vprofile
```

## Troubleshooting

Inspect application and init-container logs:

```bash
kubectl logs deployment/vproapp -n vprofile
kubectl logs deployment/vproapp -n vprofile -c init-mydb
kubectl describe pod -l app=vproapp -n vprofile
```

If the database pod is pending, check that the configured storage class can provision the `3Gi` PVC. If the ingress has no address, verify the AWS Load Balancer Controller, ACM certificate ARN, DNS record, and subnet permissions.

## Remove a Helm deployment

```bash
helm uninstall vprofile -n vprofile
kubectl delete namespace vprofile
```

Deleting the namespace also deletes namespaced resources, including the database PVC and its data according to the storage class reclaim policy.