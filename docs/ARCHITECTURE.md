## Architecture Overview

This document explains the high-level components deployed by the manifests in this repository and how they interact in a simple homelab environment.

### Components
- **Proxmox VE (out of scope here)**: Hypervisor that runs the Kubernetes nodes and ancillary VMs.
- **k3s Cluster**: Lightweight Kubernetes distribution hosting all services.
- **MetalLB (L2)**: Provides LoadBalancer IPs on a local network segment.
- **Ingress-NGINX**: Terminates HTTP(S) and routes requests to services in the cluster.
- **kube-prometheus stack**: Prometheus, Alertmanager, and Grafana for metrics and dashboards.
- **Nextcloud**: Self-hosted cloud storage application.

### Flow
1. External client resolves an internal DNS name (e.g., nextcloud.local) to the Ingress IP.
2. Traffic hits Ingress-NGINX via a MetalLB-provided LoadBalancer address.
3. Ingress routes requests to the corresponding Kubernetes Service and Pod.
4. Prometheus scrapes metrics from cluster components and applications; Grafana visualizes dashboards.


### Repository correlation
- `3-metallb/*.yaml` defines IP address pools and L2 advertisements used by Services of type LoadBalancer.
- `4-ingress-nginx/*.yaml` contains ingress objects mapping hostnames/paths to Services (e.g., Nextcloud, Grafana).
- `2-kube-prometheus.yaml` and `2-prometheus/values.yaml` configure the monitoring stack.
- `5-nextcloud/*.yaml` defines the application deployment and configuration.

### Notes and boundaries
- DNS is assumed to be provided on the home network and is not managed in this repo.
- TLS termination strategy can be handled either at Ingress or an upstream reverse proxy, depending on environment.


