# RBAC — Role-Based Access Control

## Zero-Trust Principle Applied
No service account has permissions unless explicitly granted.
Cross-namespace access is denied by default (no bindings exist across namespaces).

## Structure
| Account               | Namespace | Role              | Can Do                        |
|-----------------------|-----------|-------------------|-------------------------------|
| team-a-developer      | team-a    | developer-role    | get, list, create, update     |
| team-a-service-account| team-a    | readonly-role     | get, list, watch only         |
| team-b-developer      | team-b    | developer-role    | get, list, create, update     |
| team-b-service-account| team-b    | readonly-role     | get, list, watch only         |

## What's Blocked
- team-a cannot read/write anything in team-b namespace
- team-b cannot read/write anything in team-a namespace
- No service account can delete resources
- No service account can access Secrets

## Apply
kubectl apply -f rbac/
