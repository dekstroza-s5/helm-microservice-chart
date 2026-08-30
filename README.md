# Helm Microservice Chart

Reusable Helm chart for deploying HTTP microservices consistently across development, staging and production. It packages the repetitive Kubernetes objects around a service while leaving image, sizing, routing and application configuration configurable.

## What the chart creates

- Deployment with rolling updates, probes and resource limits
- ClusterIP Service
- optional Ingress
- ConfigMap populated from values
- optional HorizontalPodAutoscaler
- consistent labels and release ownership metadata
- configuration checksum that triggers a rollout when ConfigMap values change

## Repository layout

```text
chart/
  Chart.yaml
  values.yaml
  templates/
environments/
  dev.yaml
  prod.yaml
```

Base values define safe defaults. Environment files contain only differences, keeping reviewable configuration small.

## Prerequisites

- Kubernetes 1.27+
- Helm 3.13+
- an ingress controller when ingress is enabled
- metrics-server when autoscaling is enabled

## Static validation

```bash
helm lint chart/
helm template demo chart/ -f environments/dev.yaml > /tmp/dev.yaml
helm template demo chart/ -f environments/prod.yaml > /tmp/prod.yaml
kubectl apply --dry-run=client -f /tmp/dev.yaml
```

Example output:

```text
==> Linting chart/
1 chart(s) linted, 0 chart(s) failed
```

Inspect important rendered fields:

```bash
grep -E 'kind:|replicas:|image:|host:' /tmp/prod.yaml
```

## Development deployment

```bash
helm upgrade --install demo chart/   --namespace demo-dev   --create-namespace   -f environments/dev.yaml   --wait --timeout 2m
helm list -n demo-dev
kubectl get all,ingress -n demo-dev
```

## Production-style deployment

Always inspect the diff or rendered manifests before an upgrade:

```bash
helm upgrade --install demo chart/   --namespace demo-production   --create-namespace   -f environments/prod.yaml   --set image.repository=registry.example.com/platform/api   --set image.tag=1.4.2   --atomic --wait --timeout 5m
```

`--atomic` rolls the release back when the upgrade does not become ready within the timeout.

## Common configuration

```yaml
image:
  repository: registry.example.com/platform/api
  tag: "1.4.2"
ingress:
  enabled: true
  className: nginx
  host: api.example.com
resources:
  requests: {cpu: 200m, memory: 256Mi}
  limits: {cpu: "1", memory: 512Mi}
autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 12
  targetCPUUtilizationPercentage: 70
config:
  APP_ENV: production
  LOG_LEVEL: info
```

## Verify and rollback

```bash
helm status demo -n demo-production
helm history demo -n demo-production
kubectl rollout status -n demo-production deployment/demo-microservice
helm rollback demo 1 -n demo-production --wait
```

## Troubleshooting

- template error: run `helm lint` and `helm template --debug`.
- pods not ready: inspect events, probe paths and container logs.
- HPA has no metrics: verify metrics-server and resource requests.
- ingress returns 404: verify the host header, ingress class and Service endpoints.
- configuration changed without rollout: confirm the checksum annotation is present in the rendered Deployment.

## Uninstall

```bash
helm uninstall demo -n demo-dev
kubectl delete namespace demo-dev
```

Credentials are intentionally absent. Supply secrets through External Secrets, Sealed Secrets or the platform's secret store rather than values committed to Git.
