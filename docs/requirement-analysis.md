# Requirement Analysis — Zero-Trust Security Framework

## 1. Introduction
This document analyzes common container vulnerabilities and justifies
the Zero-Trust security model for multi-tenant Kubernetes clusters.

## 2. Common Container Vulnerabilities

### 2.1 Lateral Movement
**What it is:** Once an attacker compromises one container, they move
freely to other containers in the cluster.

**Why it happens:**
- No network segmentation between services
- Default Kubernetes allows all pod-to-pod communication
- Flat network model with no boundaries

**Real Example:** A compromised frontend pod can directly call
a database pod in another namespace with no restrictions.

**Our Mitigation:**
- Network Policies with default-deny-all
- Istio AuthorizationPolicies (whitelist only)
- Namespace isolation per tenant

### 2.2 Privilege Escalation
**What it is:** A container process gains more permissions than intended,
potentially gaining root access to the host node.

**Why it happens:**
- Containers running as root user
- Overly permissive RBAC roles
- Missing Pod Security Standards
- Writable host filesystem mounts

**Real Example:** A pod running as root can write to host paths,
escape the container, and compromise the entire node.

**Our Mitigation:**
- RBAC with least-privilege roles
- Service accounts with minimal permissions
- No cross-namespace role bindings

### 2.3 Insecure Service-to-Service Communication
**What it is:** Traffic between microservices travels unencrypted,
allowing man-in-the-middle attacks inside the cluster.

**Why it happens:**
- Kubernetes does not encrypt internal traffic by default
- HTTP used instead of HTTPS between services
- No mutual authentication between services

**Our Mitigation:**
- Istio mTLS (mutual TLS) encrypts ALL service-to-service traffic
- Both sides must present valid certificates
- Certificates auto-rotated by Istio

### 2.4 Misconfigured RBAC
**What it is:** Over-permissive roles allow services to access
resources they should never touch.

**Why it happens:**
- Using wildcard (*) permissions for convenience
- Cluster-wide roles instead of namespace-scoped roles
- No regular permission audits

**Our Mitigation:**
- Namespace-scoped roles only
- Explicit verb lists (no wildcards)
- Regular `kubectl auth can-i` audits

### 2.5 Unauthorized API Server Access
**What it is:** Direct access to the Kubernetes API server
without proper authentication.

**Why it happens:**
- Default service account tokens auto-mounted in all pods
- Tokens can be used to call the API server
- No restrictions on what the token can do

**Our Mitigation:**
- Custom service accounts with minimal roles
- automountServiceAccountToken: false where not needed

## 3. Zero-Trust Principles Applied

| Principle | Implementation |
|-----------|---------------|
| Never trust, always verify | mTLS mutual authentication |
| Least privilege access | RBAC with minimal verbs |
| Assume breach | Default-deny Network Policies |
| Explicit verification | AuthorizationPolicies whitelist |
| Audit everything | Istio access logs + audit logs |

## 4. Threat Model

### Assets to Protect
- Team A application and data
- Team B application and data
- Kubernetes API server
- Istio control plane

### Threat Actors
- Compromised container inside the cluster
- Malicious insider (developer with excess permissions)
- External attacker who breached the perimeter

### Attack Vectors Covered
- [x] Pod-to-pod lateral movement
- [x] Privilege escalation via RBAC
- [x] Unencrypted internal traffic
- [x] Unauthorized namespace access
- [x] API server abuse via service tokens

## 5. Conclusion
A Zero-Trust framework is essential for multi-tenant clusters because
the default Kubernetes security model is permissive. By combining
Network Policies, RBAC, and Istio mTLS, we create a layered security
architecture where every request is authenticated, authorized, and encrypted.
