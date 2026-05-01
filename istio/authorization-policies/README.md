# Istio Authorization Policies

## Zero-Trust Layer 3 — Request-Level Authorization

### Policy Stack
| Policy                  | Namespace     | Action | Effect                          |
|-------------------------|---------------|--------|---------------------------------|
| deny-all                | istio-system  | DENY   | Block everything mesh-wide      |
| team-a-allow-internal   | team-a        | ALLOW  | Only team-a service account     |
| team-b-allow-internal   | team-b        | ALLOW  | Only team-b service account     |
| deny-team-a-to-team-b   | team-b        | DENY   | Explicit block + audit log      |
| deny-team-b-to-team-a   | team-a        | DENY   | Explicit block + audit log      |
| allow-health-checks     | team-a,team-b | ALLOW  | Istio probe paths only          |

### Three Security Layers Combined
Layer 1 - Network Policy:     Blocks at TCP/IP level
Layer 2 - mTLS:               Blocks unauthenticated connections
Layer 3 - AuthorizationPolicy: Blocks unauthorized requests

### Apply
kubectl apply -f istio/authorization-policies/
