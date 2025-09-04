## Homelab Server Administration

This repository contains the Kubernetes manifests and Helm values I use to operate a small, personal homelab. It is intended to be transparent and reproducible for anyone curious about self-hosting on a lightweight Kubernetes distribution.

### What this repo includes
- Kubernetes namespace scaffolding
- Prometheus/Grafana monitoring via kube-prometheus
- MetalLB Layer2 load balancing configuration
- Ingress-NGINX ingress resources for internal services
- Nextcloud deployment values
- Additional self-hosted apps (Blocky DNS, Gatus, WordPress, Ghost, Redis, PostgreSQL)

### Technology overview
- **Compute**: Proxmox VE (hosted outside this repo)
- **Kubernetes**: k3s (compatible with any CNCF-conformant cluster)
- **Networking**: MetalLB (L2), Ingress-NGINX
- **Observability**: Prometheus Operator stack, Grafana
- **Applications**: Nextcloud; Blocky (DNS), Gatus (uptime), WordPress, Ghost, Redis, PostgreSQL

### CV-friendly summary (truthful)
- Administer a Proxmox VE host with a mix of VMs and containers.
- Operate a lightweight k3s cluster running multiple self-hosted applications.
- Provide internal DNS-based service discovery in the home network.
- Deploy and maintain a Nextcloud instance for reliable personal cloud storage.

No claims beyond what is represented here: uptime, node counts, and application totals vary over time in a homelab setting.

### Repository layout
```
/0-namespace.yaml                  # Core namespaces
/2-kube-prometheus.yaml            # kube-prometheus stack (Operator + CRDs)
/2-prometheus/values.yaml          # Helm values for the monitoring stack
/3-metallb.yaml                    # MetalLB setup (CRDs/controllers)
/3-metallb/*.yaml                  # Address pools and L2 advertisements
/4-ingress-nginx.yaml              # Ingress-NGINX base installation
/4-ingress-nginx/*.yaml            # Specific ingress resources (e.g., Grafana, Nextcloud, Gatus, WordPress, Ghost)
/5-nextcloud.yaml                  # Nextcloud chart installation (HelmRelease or similar)
/5-nextcloud/values.yaml           # Nextcloud configuration values
/6-apps.yaml                       # Additional Applications (Blocky, Gatus, WordPress, Ghost, Redis, PostgreSQL)
/6-*/values.yaml                   # Per-app Helm values
```

### Prerequisites
- A running Kubernetes cluster (k3s recommended)
- `kubectl` configured to talk to your cluster
- Optional: `helm` if installing charts manually

### Suggested install order
Apply manifests in a controlled order so dependencies are available when needed.

```bash
kubectl apply -f 0-namespace.yaml

# Monitoring (Prometheus Operator stack)
kubectl apply -f 2-kube-prometheus.yaml

# MetalLB for LoadBalancer services
kubectl apply -f 3-metallb.yaml
kubectl apply -f 3-metallb/pool-1.yaml
kubectl apply -f 3-metallb/l2-advertisement.yaml

# Ingress controller
kubectl apply -f 4-ingress-nginx.yaml
kubectl apply -f 4-ingress-nginx/argocd-cmd-params-cm.yaml
kubectl apply -f 4-ingress-nginx/argocd-ingress.yaml
kubectl apply -f 4-ingress-nginx/grafana-ingress.yaml
kubectl apply -f 4-ingress-nginx/nextcloud-ingress.yaml

# Nextcloud app (values used by your chosen installation method)
kubectl apply -f 5-nextcloud.yaml

# Additional apps managed by Argo CD
kubectl apply -f 6-apps.yaml
```

Notes:
- If your cluster uses a different LoadBalancer IP range, update the files under `3-metallb/` accordingly.
- The `4-ingress-nginx/*` resources assume your ingress controller and DNS are set up to resolve hostnames internally.
- The `5-nextcloud/values.yaml` file should be reviewed for storage class, persistence, and ingress hostnames.
- For additional apps under `6-*/values.yaml`, update hostnames and persistence as needed.

### Architecture
See `docs/ARCHITECTURE.md` for a high-level component diagram and explanations of how the pieces fit together.

### Security and operations
- This is a personal homelab; hardening choices are intentionally pragmatic.
- Review all manifests before applying in your environment.
- Backups, secrets management, and TLS are environment-specific and should be configured to your needs.

### License
This repository is provided under the MIT License. See `LICENSE` for details.


