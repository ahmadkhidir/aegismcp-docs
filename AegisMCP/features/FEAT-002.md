## FEAT-002: Declarative Policy-as-Code Engine for MCP Tools

**Priority:** P0
**Effort:** M
**Depends On:** FEAT-001

**User Story:**
As Marcus Vance, I want to define declarative YAML security policies governing which agents can invoke specific MCP tools and arguments, so that agents operate with strict least-privilege access.

**Description:**
A policy evaluation engine that intercepts incoming `tools/call` and `resources/read` invocations, matching the requesting agent identity and target tool against declarative security rules. Rules specify allowed, blocked, or conditional tools with argument-level constraints (such as directory whitelists, allowed HTTP domains, or maximum query row limits).

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

**Edge Cases & Failure States:**
- Syntax or schema error in a newly loaded YAML policy file → Reject policy reload, retain previously active valid policy in memory, and emit an alert to the admin log.
- Missing policy assignment for an authenticated agent identity → Default to fail-closed posture, blocking all tool executions until an explicit policy is bound.
