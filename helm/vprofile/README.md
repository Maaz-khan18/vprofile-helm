# VProfile Helm Chart

This Helm chart deploys the VProfile application stack on Kubernetes. It includes the application frontend, MySQL database, Memcached, RabbitMQ, required secret configuration, a persistent volume claim for the database, and an optional ingress for external access.

## Chart overview

The chart is located in:

- `helm/vprofile`

The current workspace also contains static Kubernetes manifests in:

- `kubedefs/`

These static manifests mirror the same components deployed by the chart but are not managed by Helm.

## Included resources

The chart creates the following Kubernetes objects:

- Deployment: `vproapp`
- Deployment: `vprodb`
- Deployment: `vpromc`
- Deployment: `vpromq01`
- Service: `vproapp-service`
- Service: `vprodb`
- Service: `vprocache01`
- Service: `vpromq01`
- Secret: `app-secret`
- PVC: `db-pv-claim`
- Ingress: `vpro-ingress` (enabled by default via values)
- Optional Docker registry secret: `docker-registry-secret`

## Folder structure

```text
helm/
└── vprofile/
    ├── Chart.yaml
    ├── README.md
    ├── values.yaml
    └── templates/
        ├── app-deployment.yaml
        ├── db-deployment.yaml
        ├── dockerregistry-secret.yaml
        ├── ingress.yaml
        ├── mc-deployment.yaml
        ├── pvc.yaml
        ├── rmq-deployment.yaml
        ├── secret.yaml
        └── services.yaml
```

## Prerequisites

- Kubernetes cluster
- Helm 3 installed
- A storage class available for the database PVC (default is `gp2` in `values.yaml`)
- Ingress controller support if `ingress.enabled` is set to `true`
- Optional: Docker registry credentials if `dockerregistry.enabled` is set to `true`

## Default values

The chart is configured with the following defaults in `values.yaml`:

## Installation

From the project root, install the chart with:

```bash
helm install vprofile ./helm/vprofile
```

If you want to create a namespace explicitly:

```bash
helm install vprofile ./helm/vprofile -n vprofile --create-namespace
```

To upgrade an existing release:

```bash
helm upgrade vprofile ./helm/vprofile
```

## Configuration notes

- The app deployment waits for the database and Memcached services before starting.
- The database uses a PVC named `db-pv-claim` and stores MySQL data under `/var/lib/mysql`.
- The app secret supplies database and RabbitMQ passwords from `secrets.dbPassword` and `secrets.rmqPassword`.
- The Ingress is configured with AWS ALB annotations by default and routes requests to `vproapp-service`.
- Docker registry credentials are only created when `dockerregistry.enabled` is set to `true`.

## Verify deployment

After install, validate the resources with:

```bash
kubectl get deploy,svc,pvc,ingress
```

To inspect the release:

```bash
helm list
helm status vprofile
```

## Notes

This chart is a basic deployment for the VProfile sample application and is intended to be customized for your cluster environment, storage class, domain, and registry credentials.
