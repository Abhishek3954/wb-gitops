[wb-code](https://github.com/Abhishek3954/Whiteboard) — application and CI/CD  
[wb-infra](https://github.com/Abhishek3954/wb-infra) — AWS infrastructure as code  
**wb-manifests**

# Kubernetes platform manifests

This repository contains the Kubernetes workloads and platform configuration for Whiteboard. Argo CD, bootstrapped by `wb-infra`, watches this repository and reconciles the platform through a Helm App-of-Apps chart.

## Repository layout

- `platform/` is the Helm chart for the root application and its platform child applications.
- `app/` contains the reusable Kustomize base for the application namespace and its web, backend, and MongoDB resources.
- `overlays/aws/` adds AWS-specific resources and settings: ALB ingress, External Secrets configuration, EBS storage, network policy, and image references.

## Platform applications

The platform chart configures six child applications in two sync waves:

1. AWS Load Balancer Controller, External Secrets Operator, and Loki.
2. The application overlay, Grafana Alloy, and kube-prometheus-stack (Prometheus and Grafana).

The `myapp` namespace uses the restricted Pod Security Standard. The frontend and backend run as non-root workloads with read-only root filesystems; the backend scales from two to four replicas and both web and backend have disruption budgets. MongoDB runs as a StatefulSet using an encrypted `gp3` EBS volume.

AWS Secrets Manager credentials are synchronized into Kubernetes by External Secrets Operator using EKS Pod Identity. The AWS Load Balancer Controller provisions an internet-facing ALB from the application Ingress. Loki and Alloy provide log aggregation, while Prometheus and Grafana provide metrics and dashboards.

## Configuration and local rendering

Terraform supplies the root chart's repository URL, revision, cluster name, AWS region, and VPC ID when it bootstraps Argo CD. Review `platform/values.yaml` and the chart templates when changing platform configuration. Application image tags and AWS-specific resources are configured in `overlays/aws/kustomization.yaml` and the other files under `overlays/aws/`.

Render or validate the application overlay with Kustomize:

```sh
kubectl kustomize overlays/aws
```

The AWS overlay expects the EKS cluster, EKS Pod Identity roles, Secrets Manager secret, and AWS Load Balancer Controller to be configured by `wb-infra` and the platform chart.
