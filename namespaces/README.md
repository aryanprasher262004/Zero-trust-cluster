# Namespaces

## Tenant Isolation Strategy
Each team runs in its own namespace. This provides:
- Resource isolation
- Independent RBAC policies
- Separate Network Policies
- Istio sidecar injection per namespace

## Namespaces
| Namespace    | Team   | Istio Injection |
|--------------|--------|-----------------|
| team-a       | Team A | Enabled         |
| team-b       | Team B | Enabled         |
| istio-system | Infra  | N/A             |

## Apply All
kubectl apply -f namespaces/
