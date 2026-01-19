# ACE13 Kubernetes Pods Project

## Project Overview

This repository contains Kubernetes manifests and supporting files for the "ACE13" example application, intended to demonstrate deployment of an application using pods, persistent volumes, ConfigMaps, Services, and Horizontal Pod Autoscaling (HPA). It is a compact, real-world-style example suitable for learning, testing, and small deployments.

Key features:
- Deployable Kubernetes manifests for app, storage, and networking
- Sample `ConfigMap` for configuration injection
- `Deployment` to manage pods and rolling updates
- `Service` to expose the application inside the cluster
- PersistentVolumeClaim and PersistentVolume example for stateful storage
- HPA configuration to autoscale based on CPU utilization

## Repository Layout

- `kubernetes/` - All Kubernetes YAML manifests
    - `configmap.yaml` - Application configuration injected into pods
    - `deployment.yaml` - `Deployment` resource describing pods, container image, env, and volumes
    - `service.yaml` - `Service` resource to expose the application (ClusterIP / NodePort / LoadBalancer as configured)
    - `pv.yaml` - Example `PersistentVolume` (hostPath/NFS provisioner depending on environment)
    - `pvc.yaml` - `PersistentVolumeClaim` used by the deployment for persistent storage
    - `hpa.yaml` - `HorizontalPodAutoscaler` configuration for autoscaling
- `bar-files/` - Binary or artifact files used by the application (example: `BVSRegFix2.bar`)

## Architecture

The application architecture is a simple containerized web/service app backed by persistent storage and managed by Kubernetes controllers.

ASCII diagram:

    +----------------------+
    |  Kubernetes Cluster  |
    |                      |
    |  +---------------+   |
    |  | Deployment    |   |
    |  | - Replicas    |<--+--- Pod(s) running container(s)
    |  | - ConfigMap   |   |   - Mounts PVC
    |  | - Volumes     |   |
    |  +---------------+   |
    |         |             |
    |         v             |
    |  +---------------+    |
    |  | Service       |----+--- Other cluster services / Ingress
    |  +---------------+    |
    |         ^              |
    |         |              |
    |  +---------------+    |
    |  | HPA           |    |
    |  +---------------+    |
    |                      |
    +----------------------+

Components:
- Pods: Run the application container(s). Managed by the `Deployment` for lifecycle and updates.
- Service: Exposes pods under a stable network endpoint.
- ConfigMap: Externalized configuration for environment variables or config files.
- PVC/PV: Persistent storage for application state.
- HPA: Scales the `Deployment` replicas based on CPU (or other metrics) thresholds.

## Prerequisites

- A Kubernetes cluster (minikube, kind, cloud provider cluster, or on-prem)
- `kubectl` configured to connect to the target cluster
- Optional: `helm` if you prefer templating (not required here)

Tested with Kubernetes v1.20+ but manifests are compatible with most recent releases.

## Deployment Instructions

1. Review and customize manifests in the `kubernetes/` directory.

2. Apply resources in recommended order:

```bash
# from repository root
kubectl apply -f kubernetes/pv.yaml
kubectl apply -f kubernetes/pvc.yaml
kubectl apply -f kubernetes/configmap.yaml
kubectl apply -f kubernetes/deployment.yaml
kubectl apply -f kubernetes/service.yaml
kubectl apply -f kubernetes/hpa.yaml
```

3. Verify resources:

```bash
kubectl get pods
kubectl get deploy
kubectl get svc
kubectl get pvc
kubectl get pv
kubectl get hpa
```

4. Inspect logs for troubleshooting:

```bash
kubectl logs -l app=ace13
# or for a specific pod
kubectl logs <pod-name>
```

Notes:
- If `pv.yaml` uses `hostPath`, it is suitable only for single-node or development clusters.
- For production, provide a cloud or network-backed storage class and provisioner.

## Configuration

- Environment variables and config files are provided via `kubernetes/configmap.yaml` and referenced by `kubernetes/deployment.yaml`.
- To change container image or resource requests/limits, edit `kubernetes/deployment.yaml` fields:
    - `spec.template.spec.containers[].image`
    - `resources.requests` and `resources.limits`
    - Ports and volume mounts

## Scaling and HPA

- The HPA (`kubernetes/hpa.yaml`) is configured to watch the `Deployment` by name and scale pods based on CPU utilization thresholds. Adjust `minReplicas`, `maxReplicas`, and `targetCPUUtilizationPercentage` to fit your workload.

To manually scale the deployment:

```bash
kubectl scale deployment <deployment-name> --replicas=3
```

## Common Tasks

- Get pod details and events:

```bash
kubectl describe pod <pod-name>
kubectl describe deployment <deployment-name>
kubectl describe pvc <pvc-name>
```

- Port-forward service for local testing:

```bash
kubectl port-forward svc/<service-name> 8080:80
# then open http://localhost:8080
```

- Delete all deployed resources (cleanup):

```bash
kubectl delete -f kubernetes/hpa.yaml
kubectl delete -f kubernetes/service.yaml
kubectl delete -f kubernetes/deployment.yaml
kubectl delete -f kubernetes/configmap.yaml
kubectl delete -f kubernetes/pvc.yaml
kubectl delete -f kubernetes/pv.yaml
```

## Troubleshooting

- Pods stuck in `Pending`: check PVC binding status and PV availability and node selectors/taints.
- CrashLoopBackOff: inspect `kubectl logs` and `kubectl describe pod` for container errors and readiness/liveness probes.
- HPA not scaling: confirm metrics server is installed (`kubectl top nodes`), HPA references the correct deployment and metrics.

## Security and Best Practices

- Do not store secrets in `ConfigMap`; use `Secret` resources for sensitive data and mount them as environment variables or volumes.
- For production, use resource limits and requests to protect node capacity.
- Use RBAC to restrict access to cluster resources.

## Extending this Project

- Convert manifests to Helm charts for templating and environment-specific values.
- Add an `Ingress` resource with TLS termination for external access.
- Integrate CI/CD pipeline to build/push container images and automatically apply manifests to the target cluster.

## Files of Interest

- `kubernetes/deployment.yaml` - deployment specification and pod template
- `kubernetes/service.yaml` - service to expose the app
- `kubernetes/configmap.yaml` - app configuration
- `kubernetes/pv.yaml` / `kubernetes/pvc.yaml` - persistent storage example
- `kubernetes/hpa.yaml` - autoscaling rules

## Contact

For questions or contributions, open an issue or submit a pull request.

---

Generated README for ace13-kubernetes-pods-project
