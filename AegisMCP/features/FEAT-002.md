## FEAT-002: Declarative Policy-as-Code Engine for MCP Tools

**Priority:** P0
**Effort:** M
**Depends On:** FEAT-001

**User Story:**
As Marcus Vance, I want to define declarative YAML security policies governing which agents can invoke specific MCP tools and arguments, so that agents operate with strict least-privilege access.

**Description:**
A policy evaluation engine that intercepts incoming `tools/call` and `resources/read` invocations, matching the requesting agent identity and target tool against declarative security rules. Rules specify allowed, blocked, or conditional tools with argument-level constraints (such as directory whitelists, allowed HTTP domains, or maximum query row limits).

Policy versions follow one lifecycle in every deployment. A version is published as a draft, which is stored and readable through the catalogue but never governs traffic. Activation is the only event that changes what the gateway enforces, and it replaces the compiled policy atomically: either the whole document is valid and compiled, or the previously active version remains in force.

Deployment configuration selects the activation path. In a managed deployment an authorized administrator, or a CI/CD service account holding the administrator role, activates a version from the web console or an automated pipeline. In a GitOps-managed deployment the policy document delivered with the deployment is the sole source of truth: the gateway applies it at startup, watches it for content changes, reloads a changed document without a restart, and records the commit that introduced it. Activation from the console or a pipeline is refused in that mode, and the console directs the operator to open a pull request instead.

Activation also states the version it replaces, so two operators cannot silently overwrite each other's deployment. Activating a version archives the version it replaced, and a document identical to a published draft activates that draft record rather than creating a duplicate.

**Acceptance Criteria:**
- **Given** a loaded security policy with an explicit tool whitelist
  **When** an agent calls a tool present in the whitelist with compliant arguments
  **Then** the request is marked as allowed and forwarded to the target tool server.
- **Given** an agent calling a tool not explicitly permitted in its assigned policy
  **When** the invocation is evaluated by the policy engine
  **Then** the tool call is blocked immediately, returning a permission denial error without notifying the upstream tool server.
- **Given** a policy specifying argument constraints (e.g. `path` must start with `/workspace`)
  **When** an agent submits an argument violating the constraint (e.g. `path: "/etc/passwd"`)
  **Then** the request is blocked and an argument violation event is logged.
- **Given** a published draft policy version
  **When** an agent invokes a tool
  **Then** the draft does not affect the decision and the active version evaluates the call.
- **Given** an authorized operator activating a valid version whose stated base version is still active
  **When** the activation completes
  **Then** every subsequent tool call evaluates against the new version, the replaced version is archived, and the response identifies the newly active version.
- **Given** a GitOps-managed deployment whose policy document changes in the repository and is delivered to the gateway
  **When** the document is reloaded
  **Then** the gateway enforces the changed policy without a restart and records the commit that introduced it.
- **Given** a GitOps-managed deployment
  **When** an operator or pipeline attempts a direct activation
  **Then** the attempt is refused, the active policy remains in force, and the refusal is recorded.
- **Given** an activation whose stated base version is no longer active
  **When** the activation is evaluated
  **Then** it is rejected as a conflict, the active policy remains in force, and no version changes state.
- **Given** an accepted or rejected publish, activation, or GitOps reload
  **When** the action completes
  **Then** an administrative audit entry is sealed with the acting principal (the commit hash for a reload), the policy name and version, the outcome, and the reason category on rejection, and it is retained for the 365-day ledger period.

**Edge Cases & Failure States:**
- Syntax or schema error in a published or reloaded policy document → Reject the change, retain the previously active valid policy in force, and emit an alert to the admin log.
- GitOps-managed policy document missing, unreadable, or invalid at startup → Start with no active policy, deny every tool call, and raise an administrative alert; never fall back to a draft.
- Missing policy assignment for an authenticated agent identity → Default to fail-closed posture, blocking all tool executions until an explicit policy is bound.
- No policy version active yet on a fresh deployment → Accept an activation with an empty base version; every tool call is denied until the first activation succeeds.
- Two activations racing from the same base version → The first succeeds, the second is rejected as a conflict, and both attempts are audited.
