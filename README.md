# Helm Microservice Chart

Reusable Helm chart for deploying HTTP microservices consistently across development, staging, and production.

## Features

- Deployment, Service, Ingress and autoscaling
- configurable probes and resource limits
- ConfigMap-driven settings
- optional ServiceAccount
- environment-specific values files

## Validate

```bash
helm lint chart/
helm template demo chart/ -f environments/dev.yaml
```

## Install

```bash
helm upgrade --install demo chart/ --namespace demo --create-namespace -f environments/dev.yaml
```

Supply production secrets through a secret manager. Do not commit credentials.
