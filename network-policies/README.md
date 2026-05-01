# Network Policies

## Zero-Trust Network Strategy

### Default Deny-All
Every namespace starts with a deny-all policy.
Nothing can enter or leave unless explicitly permitted.

### Allow Rules
| Policy            | Namespace | What it allows                  |
|-------------------|-----------|---------------------------------|
| default-deny-all  | team-a    | Blocks everything by default    |
| default-deny-all  | team-b    | Blocks everything by default    |
| allow-internal    | team-a    | Pods within team-a only         |
| allow-internal    | team-b    | Pods within team-b only         |
| allow-dns         | team-a    | DNS port 53 (UDP+TCP)           |
| allow-dns         | team-b    | DNS port 53 (UDP+TCP)           |

### What Is Blocked
- team-a → team-b (cross-namespace) BLOCKED
- team-b → team-a (cross-namespace) BLOCKED
- External internet → team-a BLOCKED
- External internet → team-b BLOCKED

## Apply All
kubectl apply -f network-policies/
