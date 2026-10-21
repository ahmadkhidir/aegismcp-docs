## FEAT-012: Dynamic Argument Schema Validator & Sanitizer

**Priority:** P2
**Effort:** M
**Depends On:** FEAT-002

**User Story:**
As David Chen, I want tool parameters to be automatically validated against strict JSON schema definitions with SQL-injection and path-traversal sanitization, so that flawed agent outputs do not corrupt downstream systems.

**Description:**
A pre-execution payload validation and sanitization filter. It retrieves the published JSON schema for the target MCP tool, validates all parameter keys and data types against the schema, and runs specialized heuristics to detect SQL injection keywords, command injection tokens (`&&`, `|`, `;`), and directory traversal patterns (`../`, `..\`) in string arguments.

**Acceptance Criteria:**
- **Given** an agent invoking a tool with arguments that deviate from the tool's published JSON schema (e.g. string passed instead of integer)
  **When** evaluated by the schema validator
  **Then** the tool call is rejected before reaching the server, returning a detailed schema validation error to the agent.
- **Given** an agent submitting a filesystem path argument containing directory traversal (e.g. `path: "/allowed/dir/../../etc/shadow"`)
  **When** inspected by the path sanitizer
  **Then** the path is resolved to its canonical form, identified as an out-of-bounds traversal, and blocked.
- **Given** a database query tool receiving a parameter with detected SQL injection payloads
  **When** analyzed by the sanitizer
  **Then** the request is blocked and logged as an untrusted parameter violation.

**Edge Cases & Failure States:**
- Tool published schema is empty or missing from MCP server manifest → Fall back to generic strict type checks and basic command injection heuristics.
- Intentionally complex legitimate SQL queries passed by authorized database administration agents → Support policy overrides for specific whitelisted tool identities.
