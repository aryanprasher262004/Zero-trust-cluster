# Zero-Trust Architecture Documentation

## 1. System Overview

This project implements a Zero-Trust Security Framework on a local
Kubernetes cluster using Docker Desktop. The architecture ensures
no service is trusted by default and every request is explicitly
authenticated, authorized, and encrypted.
┌─────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                    │
│                                                         │
│  ┌─────────────────┐      ┌─────────────────┐          │
│  │   Namespace      │      │   Namespace      │          │
│  │    team-a        │      │    team-b        │          │
│  │                 │      │                 │          │
│  │  ┌───────────┐  │      │  ┌───────────┐  │          │
│  │  │ team-a-app│  │  ✖   │  │ team-b-app│  │          │
│  │  │  (nginx)  │  │◄────►│  │  (nginx)  │  │          │
│  │  │           │  │BLOCK │  │           │  │          │
│  │  └─────┬─────┘  │      │  └─────┬─────┘  │          │
│  │        │sidecar │      │        │sidecar │          │
│  │  ┌─────▼─────┐  │      │  ┌─────▼─────┐  │          │
│  │  │istio-proxy│  │      │  │istio-proxy│  │          │
│  │  └───────────┘  │      │  └───────────┘  │          │
│  └─────────────────┘      └─────────────────┘          │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │              istio-system namespace              │   │
│  │   istiod (CA) │ ingress-gateway │ egress-gateway │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘

## 2. Security Layers

### Layer 1 — RBAC (Role-Based Access Control)
- Every team has its own ServiceAccount
- Roles are namespace-scoped (no cluster-wide roles)
- Least privilege: only get, list, watch, create, update
- No cross-namespace role bindings exist
- Verified with: kubectl auth can-i

### Layer 2 — Network Policies
- Default deny-all applied to every namespace
- Only intra-namespace traffic allowed
- DNS (port 53) explicitly allowed
- Cross-namespace traffic blocked at TCP/IP level
- Verified with: wget timeout between namespaces

### Layer 3 — Istio mTLS (Mutual TLS)
- PeerAuthentication mode: STRICT in all namespaces
- Both client and server must present valid certificates
- Certificates issued and rotated by Istio CA (istiod)
- DestinationRule enforces ISTIO_MUTUAL on all clients
- Verified with: istioctl x describe pod

### Layer 4 — Istio Authorization Policies
- Mesh-wide deny-all as baseline
- Explicit ALLOW only for same-namespace service accounts
- Explicit DENY for cross-namespace with audit logging
- Health check paths whitelisted separately
- Verified with: istio-proxy logs showing DC disconnect

## 3. Component Details

### Namespaces
| Namespace    | Purpose          | Istio Injection |
|--------------|------------------|-----------------|
| team-a       | Tenant A workload| Enabled         |
| team-b       | Tenant B workload| Enabled         |
| istio-system | Service mesh CP  | N/A             |

### RBAC Structure
| ServiceAccount        | Namespace | Role           |
|-----------------------|-----------|----------------|
| team-a-service-account| team-a    | readonly-role  |
| team-a-developer      | team-a    | developer-role |
| team-b-service-account| team-b    | readonly-role  |
| team-b-developer      | team-b    | developer-role |

### Network Policies
| Policy           | Namespace | Effect                    |
|------------------|-----------|---------------------------|
| default-deny-all | team-a    | Block all ingress + egress|
| default-deny-all | team-b    | Block all ingress + egress|
| allow-internal   | team-a    | Allow within namespace    |
| allow-internal   | team-b    | Allow within namespace    |
| allow-dns        | both      | Allow port 53             |

### Istio Policies
| Policy                | Type                | Effect              |
|-----------------------|---------------------|---------------------|
| mesh-wide-mtls        | PeerAuthentication  | STRICT mesh-wide    |
| team-a-mtls           | PeerAuthentication  | STRICT team-a       |
| team-b-mtls           | PeerAuthentication  | STRICT team-b       |
| deny-all              | AuthorizationPolicy | Deny mesh-wide      |
| team-a-allow-internal | AuthorizationPolicy | Allow team-a only   |
| team-b-allow-internal | AuthorizationPolicy | Allow team-b only   |
| deny-team-a-to-team-b | AuthorizationPolicy | Explicit cross deny |
| deny-team-b-to-team-a | AuthorizationPolicy | Explicit cross deny |

## 4. Data Flow

### Allowed Flow (within namespace)
team-a-app ──mTLS──► istio-proxy ──verify cert──► team-a-service
│                    │                              │
│              check AuthPolicy                     │
│              ALLOW (same ns)                      │
└────────────────────────────────────────────────►  ✅

### Blocked Flow (cross namespace)
team-a-app ──► istio-proxy ──► Network Policy DENY ──► TIMEOUT
│
└──► AuthorizationPolicy DENY ──► DC disconnect

## 5. Tools Used
| Tool          | Version  | Purpose                    |
|---------------|----------|----------------------------|
| Docker Desktop| Latest   | Local Kubernetes cluster   |
| Kubernetes    | v1.28+   | Container orchestration    |
| Istio         | v1.21+   | Service mesh + mTLS        |
| kubectl       | v1.28+   | Cluster management         |
| istioctl      | v1.21+   | Istio management + verify  |
| Git/GitHub    | Latest   | Version control + collab   |

## 6. Observability — Kiali Dashboard

Kiali provides a visual graph of the service mesh showing
mTLS connections and blocked traffic in real time.

### Install Kiali
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.21/samples/addons/kiali.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.21/samples/addons/prometheus.yaml

### Access Dashboard
istioctl dashboard kiali

### What to Look For
- Green locks on connections = mTLS active
- Red lines = blocked/denied traffic
- Graph shows team-a and team-b as isolated islands
