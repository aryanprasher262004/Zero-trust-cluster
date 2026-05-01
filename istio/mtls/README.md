# Istio mTLS Configuration

## What is mTLS?
Mutual TLS requires BOTH the client and server to present
valid certificates before any communication is allowed.

## Mode: STRICT
STRICT mode means:
- Plain HTTP traffic is REJECTED
- Both sides must have valid Istio-issued certificates
- Certificates are auto-rotated by Istio every 24 hours

## Files
| File                  | Purpose                              |
|-----------------------|--------------------------------------|
| team-a-mtls.yaml      | Enforce mTLS in team-a namespace     |
| team-b-mtls.yaml      | Enforce mTLS in team-b namespace     |
| mesh-wide-mtls.yaml   | Baseline mTLS for entire mesh        |
| destination-rules.yaml| Tell clients to send mTLS traffic    |

## Verify mTLS is Active
istioctl authn tls-check <pod-name>.<namespace>

## Apply
kubectl apply -f istio/mtls/
