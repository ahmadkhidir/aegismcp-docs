## FEAT-003: Local Developer CLI Proxy (`aegis-cli`)

**Priority:** P1
**Effort:** M
**Depends On:** FEAT-001, FEAT-002

**User Story:**
As David Chen, I want to run a local CLI wrapper that intercepts MCP connections from my IDE/desktop agent and enforces local policy files, so that I can validate security guardrails and debug tool arguments before deploying to cloud.

**Description:**
A standalone command-line executable (`aegis-cli`) designed for developers and security testing. It wraps existing local MCP stdio or SSE servers (e.g. `aegis-cli wrap --policy ./aegis.yaml -- npx @modelcontextprotocol/server-filesystem /tmp`), prints real-time color-coded terminal inspection logs of all JSON-RPC tool calls, and reports policy pass/fail statuses.

**Acceptance Criteria:**
- **Given** a local terminal environment with `aegis-cli` installed
  **When** a developer executes `aegis-cli wrap --policy policy.yaml -- <mcp-server-command>`
  **Then** the CLI starts the upstream server as a managed child process and exposes wrapped stdio streams to the parent client.
- **Given** an active local debugging session
  **When** an agent client invokes a tool through the wrapped process
  **Then** the terminal renders an interactive, formatted log showing tool name, arguments, evaluation time, and policy decision.
- **Given** a developer checking a policy file for validity
  **When** running `aegis-cli lint policy.yaml`
  **Then** the command validates schema compliance, detects unreachable rules, and returns exit code 0 or detailed syntax error messages.

**Edge Cases & Failure States:**
- Child MCP server exits unexpectedly → Cleanly terminate stdio pipes, output the child process exit code, and exit with status 1.
- Incompatible command arguments passed to wrapper → Print usage instructions and flag errors without launching orphan child processes.
