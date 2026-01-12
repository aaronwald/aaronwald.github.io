---
layout: post
title: "Homelab Infrastructure Overview"
date: 2026-01-12 00:00:00 -0500
categories: homelab infrastructure
---

A 3-node Kubernetes homelab built for learning, running personal services, and having a platform to experiment with.

![Homelab rack](/assets/images/homelab-rack.jpg)

## Hardware

Three Minisforum mini PCs in a 3D-printed rack:

- **1x UM760** (Ryzen 5 7640HS) - 32GB RAM, 1TB NVMe
- **2x UM870** (Ryzen 7 8745H) - 32GB RAM, 1TB NVMe each

Small, quiet, power-efficient. The yellow rack keeps them organized with a 4-port KVM switch for management and a JetKVM for remote access.

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

No inbound ports opened on the router. All external access goes through Tailscale or Cloudflare tunnels.

## Key Decisions

**Why K3s over full K8s?** Lower resource overhead, built-in components (CoreDNS, Traefik), single binary. Perfect for homelab scale.

**Why Flux over ArgoCD?** Lighter weight, pull-based GitOps, good Helm support. Works well with the "git push and walk away" workflow.

**Why Pi-hole outside the cluster?** DNS is too critical to depend on the thing it's providing DNS for. Keeps infrastructure bootstrapping simple.

**Why Longhorn?** Native K8s storage that just works. 3-way replication across nodes means a node can die without data loss.

## Current State

- ~10 HelmReleases managed by Flux
- Brooklyn, NY backup site with Garage S3 for off-site Velero backups
- APC UPS monitoring via Prometheus
- Zero manual kubectl applies - everything through git

## What's Next

- Fourth Proxmox node for more headroom
- Falco for runtime security monitoring
- More automation around disaster recovery testing

---

*Last updated: January 2026*
