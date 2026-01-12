---
layout: post
title: "Homelab Infrastructure Overview"
date: 2026-01-12 00:00:00 -0500
categories: homelab infrastructure
---

A 3-node Kubernetes homelab. Been meaning to post this for a while.

![Homelab rack](/assets/images/homelab-rack.jpg)

## Hardware

Three Minisforum mini PCs in a 3D-printed rack:

- **1x UM760** (Ryzen 5 7640HS) - 32GB RAM, 1TB NVMe
- **2x UM870** (Ryzen 7 8745H) - 32GB RAM, 1TB NVMe each

Small, quiet, power-efficient. The yellow rack is the [Modular 10" Server Rack (MOD10)](https://www.printables.com/model/1225275-modular-10-server-rack-mod10) by Mandic., with an [EZCOO 4-port KVM + JetKVM setup](https://medium.com/@cloudbree/jetkvm-ezcoo-4x1-hotkey-switching-working-tutorial-3d8ac6d7d18e) for local and remote management.

## Stack

```
Proxmox VE 8 (hypervisor)
    └── K3s 1.28 (lightweight Kubernetes)
         └── Flux CD 2.x (GitOps)
              └── All workloads deployed via git
```

Everything is infrastructure-as-code: OpenTofu for VM provisioning, Ansible for configuration, Flux for Kubernetes resources. Git is the single source of truth.

## Core Services

All deployed as Flux HelmReleases:

| Service | Purpose |
|---------|---------|
| **Traefik** | Ingress controller, TLS termination |
| **Longhorn** | Distributed storage with 3-way replication |
| **cert-manager** | Automatic Let's Encrypt certificates |
| **Authentik** | SSO/identity provider (OIDC) |
| **Velero** | Kubernetes backup to Google Cloud Storage |
| **MetalLB** | Bare-metal LoadBalancer |
| **Tailscale** | VPN mesh for remote access |

Observability stack: Prometheus + Grafana + Loki for metrics, dashboards, and logs.

## Networking

- **Pi-hole VM** outside K8s handles DNS and ad-blocking for the homelab VLAN
- **Tailscale** provides secure remote access from anywhere
- **Cloudflare Tunnel** exposes select services publicly without port forwarding
- **UniFi Dream Machine** as the network gateway with dedicated homelab VLAN

---

*Last updated: January 2026. Written with [Claude](https://claude.ai).*
