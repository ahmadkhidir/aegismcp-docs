## Screen 0: Enterprise SSO / OIDC Login

**Route:** `/login` (Initial Authentication Gate)
**Platform:** Web
**Presentation:** Page
**Purpose:** Authenticate human security and platform administrators accessing the Web Management Console via corporate Single Sign-On (SSO / OIDC), enforce Multi-Factor Authentication (MFA), and establish role-scoped access control ([FEAT-008](../features/FEAT-008.md), [FEAT-009](../features/FEAT-009.md), [NFR-029](../nfr.md)).
**Visual Focus:** Centered brand shield mark with high-visibility "Sign in with Enterprise SSO" primary action.
**Action Hierarchy:** Primary: Authenticate via Corporate Identity Provider (OIDC / SAML); Secondary: Select Development/Staging Persona (when non-production mock mode is active).
**Entry Points:** Initial console navigation while unauthenticated, session timeout redirect ([NFR-029](../nfr.md)), or explicit sign-out.
**Exit Points:** [Security Dashboard](01-dashboard.md) (or originally requested destination route upon successful authentication).

### Layout

**Web layout:**
- Centered authentication card container on deep dark neutral canvas (`grey-50`).
- Header: AegisMCP hexagonal shield brand mark (`A-002`), title "AegisMCP", and version badge.
- Subtitle: "Zero-Trust Reverse Proxy & Security Firewall for Model Context Protocol".
- Primary authentication panel:
  - Corporate SSO / OIDC trigger button initiating Authorization Code Flow with PKCE.
  - Active Identity Provider status (e.g. connected enterprise IdP issuer or local staging provider).
- Development / Staging Persona Switcher (rendered when running in non-production environments):
  - Fast-login buttons for canonical personas: Elena Rostova (Admin), Priya Sharma (Auditor), Marcus Vance (Operator), David Chen (Viewer).
- Security compliance footer: "NFR-029 & FEAT-009 Zero-Trust Identity Compliant".

### Components

| Component | Type | Platform | Purpose | Data Source |
|---|---|---|---|---|
| Brand Header | Header | Web | Identifies platform, version, and mission | Static asset `A-002` |
| SSO Action Panel | Card | Web | Triggers enterprise IdP redirect with PKCE challenge | OIDC Client Configuration |
| Persona Selector | Grid | Web | Allows instant role-switching for local evaluation and testing | Persona directory ([user-personas](../user-personas.md)) |
| Security Footer | Footer | Web | Displays governance and cryptographic identity standards compliance | Static text |

### Overlays

None (this screen acts as the root authentication gate overlaying the application shell).

### Data Requirements

**Data Needed:**
- Enterprise Identity Provider configuration (Issuer URL, Client ID, Redirect URI, Scopes).
- Stored session state and cryptographic PKCE verifier parameters (`state`, `nonce`, `code_challenge`).

**Data Flow:**
1. Screen mounts → verify if active token session exists in secure storage.
2. If authenticated session exists and has not expired → immediately redirect to requested console route.
3. If unauthenticated → render SSO action panel.
4. User clicks "Sign in with Enterprise SSO" → generate cryptographically random PKCE `code_verifier` (SHA-256 challenge) and redirect browser to IdP authorization endpoint (`/protocol/openid-connect/auth`).
5. IdP authenticates user (with MFA) and returns authorization `code` to redirect URI.
6. Exchange authorization `code` at token endpoint for ID Token and Access Token; decode claims and extract RBAC roles (`admin`, `operator`, `auditor`, `viewer`).
7. Persist session with 15-minute inactivity tracker ([NFR-029](../nfr.md)) and transition to [Security Dashboard](01-dashboard.md).

### States

| State | Condition | What the User Sees |
|---|---|---|
| **Default (Unauthenticated)** | No valid session active | Clean login card with primary SSO action and persona shortcuts in non-production |
| **Authenticating / Redirecting** | Authorization handshake in flight | Spinner indicator with text "Redirecting to Corporate Identity Provider..." |
| **Authentication Error** | Invalid token signature, expired code, or state mismatch | Error alert banner with retry action and security log correlation ID |
| **Access Denied (Insufficient Role)** | Authenticated user lacks authorized console roles | Warning banner indicating account lacks `admin`, `operator`, `auditor`, or `viewer` entitlement |
