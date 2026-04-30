# Zero-trust-cluster
# Zero-Trust Security Framework for Multi-Tenant Kubernetes Clusters

## Overview
This project implements a Zero-Trust security model on a local Kubernetes cluster
using Istio Service Mesh, RBAC, and Network Policies.

## Architecture
- **Tenants**: team-a, team-b (isolated namespaces)
- **Service Mesh**: Istio with mutual TLS (mTLS)
- **Access Control**: RBAC + Network Policies (default-deny)
- **Observability**: Kiali + audit logs

## Prerequisites
| Tool | Version | Install |
|------|---------|---------|
| Docker Desktop | Latest | https://docker.com/products/docker-desktop |
| kubectl | v1.28+ | Included with Docker Desktop |
| istioctl | 1.20+ | See setup below |
| Git | Any | https://git-scm.com |

## Quick Setup
```bash
git clone https://github.com/YOUR_USERNAME/zero-trust-cluster.git
cd zero-trust-cluster
```
Then follow the phase guides in `/docs`.

## Project Structure
zero-trust-cluster/
├── docs/                        # Architecture & analysis docs
├── namespaces/                  # Tenant namespace definitions
├── rbac/                        # Roles and RoleBindings
├── network-policies/            # Default-deny + allow rules
├── istio/
│   ├── mtls/                    # PeerAuthentication configs
│   └── authorization-policies/  # AuthorizationPolicy configs
├── sample-apps/                 # Demo apps for team-a, team-b
└── audit/                       # Security logs and reports
