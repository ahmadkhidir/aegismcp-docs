## FEAT-009: Agent Identity Verification & SPIFFE/OIDC Integration

**Priority:** P1
**Effort:** M
**Depends On:** FEAT-001, FEAT-002

**User Story:**
As Marcus Vance, I want AegisMCP to authenticate incoming agent connections using SPIFFE/SPIRE cryptographic identities or OIDC service tokens, so that tool policies are enforced per verified agent identity rather than ambient network location.

**Description:**
An identity authentication module that validates cryptographic credentials presented by client agents during connection establishment. It supports SPIFFE/SPIRE x509 SVIDs (mTLS) for Kubernetes workloads, OAuth 2.0 / OpenID Connect Bearer JWTs, and API tokens, mapping verified identities to internal security policy domains.

**Acceptance Criteria:**
- **Given** a Kubernetes pod running an AI agent with an injected SPIFFE Workload API socket
  **When** the agent initiates an mTLS connection to AegisMCP
  **Then** the gateway validates the SVID certificate chain, extracts the SPIFFE ID (e.g. `spiffe://cluster.local/ns/agents/sa/support-bot`), and binds the session to that exact identity.
- **Given** an agent presenting an OIDC Bearer token signed by the enterprise IdP
  **When** connecting to the gateway over HTTP/SSE
  **Then** AegisMCP validates the JWT signature, issuer, and expiry against configured JWKS, extracting role claims for policy evaluation.
- **Given** a client attempting connection with an expired, revoked, or untrusted credential
  **When** the handshake occurs
  **Then** the connection is terminated immediately with HTTP 401 Unauthorized or TLS handshake failure.

**Edge Cases & Failure States:**
- OIDC IdP JWKS endpoint unreachable during key rotation → Use cached public keys within grace period; if key is unrecognized, reject token and emit identity provider error.
- Clock skew between agent client and gateway server → Enforce standard 60-second leeway on JWT `nbf` and `exp` validations.
