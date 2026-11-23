## FEAT-003: Local Developer CLI Proxy (`aegis-cli`)

**Priority:** P1
**Effort:** M
**Depends On:** FEAT-001, FEAT-002

**User Story:**
As David Chen, I want to run a local CLI wrapper that intercepts MCP connections from my IDE/desktop agent and enforces local policy files, so that I can validate security guardrails and debug tool arguments before deploying to cloud.

**Description:**
A standalone command-line executable for developers and security testing. It wraps a local MCP stdio server as a managed child process, prints real-time color-coded inspection logs of every intercepted JSON-RPC call, and reports the policy decision for each. The wrapper manages a child process only: an SSE or WebSocket upstream is reached through the gateway, never through `wrap`.

The command surface is:
- `aegis-cli wrap --policy <file> [--no-color] -- <command> [args...]` compiles the policy once at start, then relays the agent's standard I/O through policy evaluation and DLP scrubbing to the child process, using the stdio contract defined in [FEAT-001](FEAT-001.md). Each inspection line on standard error carries the time, tool name, arguments with every value truncated to 512 characters and credential- or PII-shaped values replaced, the decision, and the evaluation duration. Color is used only when standard error is a terminal and `--no-color` is absent. No gateway connection is made.
- `aegis-cli lint <file>` validates the policy schema, argument constraints, and rule reachability, and reports shadowed or unreachable rules as warnings without failing.
- `aegis-cli kill (--agent <id> | --target-type <session|agent|server|global> [--target-id <id>]) [--reason <text>] [--gateway <url>] [--token <jwt>] [--second-approver <operator>] [--confirm-lockdown]` sends the emergency action, prints the returned status, affected session count, and timestamp, and reports every refusal with its HTTP status and reason. `--agent` is shorthand for the `agent` target type. A `global` target requires `--confirm-lockdown` and sends no `targetId`.
- `aegis-cli verify-audit --ledger <file> [--from <block-index>]` recomputes the ledger's SHA-256 chain from the genesis block, or from the given index using its predecessor's sealed hash, to the sealed head, and reports the block count and head hash. It names the first index whose block hash, parent link, or sealed coverage fails to verify ([NFR-026](../nfr.md)).

Exit codes mean the same thing for every command: `0` success, `1` a failure the command ran into (invalid policy, broken ledger, refused or failed gateway request, child failure, or internal error), `2` a usage error or unreadable input path, `130` after an interrupt, `143` after a termination signal. Standard output carries results for `lint`, `kill`, and `verify-audit`; `wrap` reserves standard output for the protocol stream.

**Acceptance Criteria:**
- **Given** a local terminal environment with `aegis-cli` installed
  **When** a developer executes `aegis-cli wrap --policy policy.yaml -- <mcp-server-command>`
  **Then** the CLI starts the upstream server as a managed child process and exposes wrapped stdio streams to the parent client.
- **Given** an active local debugging session
  **When** an agent client invokes a tool through the wrapped process
  **Then** the terminal renders an interactive, formatted log showing tool name, arguments, evaluation time, and policy decision.
- **Given** a wrapped session running without `--no-color`
  **When** inspection lines are written
  **Then** they go to standard error, standard output carries only JSON-RPC messages, and a value longer than 512 characters or shaped like a credential is never printed in full.
- **Given** a developer checking a policy file for validity
  **When** running `aegis-cli lint policy.yaml`
  **Then** the command validates schema compliance, detects unreachable rules, and exits 0 with a warning line for each unreachable rule.
- **Given** an invalid policy file
  **When** running `aegis-cli lint policy.yaml`
  **Then** the command exits 1 and prints the file, line, and column of the first error to standard error.
- **Given** an unreadable or missing policy path
  **When** running `aegis-cli lint` or `aegis-cli wrap`
  **Then** the command exits 2 without launching a child process.
- **Given** an accepted emergency action
  **When** `aegis-cli kill` receives the gateway response
  **Then** it prints the returned status, affected session count, and timestamp to standard output and exits 0.
- **Given** a `global` target without `--confirm-lockdown`
  **When** the operator runs `aegis-cli kill`
  **Then** the CLI exits 2, sends no request, and prints why the lockdown was refused.
- **Given** a `global` target with `--confirm-lockdown`
  **When** the operator runs `aegis-cli kill`
  **Then** the request carries `targetType` `global`, no `targetId`, and the `--second-approver` value when one was supplied.
- **Given** the gateway answers `401`, `403`, `404`, or `503` to a kill request
  **When** the CLI receives it
  **Then** the CLI exits 1 and prints the status and reason to standard error; a `400` answer exits 2.
- **Given** a ledger whose chain is intact
  **When** an auditor runs `aegis-cli verify-audit --ledger ledger.wal`
  **Then** the command prints the verified block count and head hash and exits 0.
- **Given** a ledger with a modified, reordered, or missing block
  **When** an auditor runs `aegis-cli verify-audit --ledger ledger.wal`
  **Then** the command exits 1 and names the first index that fails verification.
- **Given** an unknown option, a missing required option, or a missing `--` separator
  **When** any command is run
  **Then** the CLI prints usage to standard error, writes nothing to standard output, and exits 2.

**Edge Cases & Failure States:**
- Child MCP server exits unexpectedly → Cleanly terminate stdio pipes, output the child process exit code, and exit with status 1.
- Incompatible command arguments passed to wrapper → Print usage instructions and flag errors without launching orphan child processes.
- The agent closes its input while calls are still in flight → Stop reading input, let the in-flight calls finish, close the child's input, and exit 0 when the child exits 0.
- Interrupt or termination signal during a wrapped session → Forward the signal to the child, wait up to five seconds for it to exit, terminate it if it is still running, and exit 130 or 143.
- Standard error is closed by the caller → Continue proxying the protocol stream and drop inspection lines rather than writing them to standard output.
- Gateway unreachable during a kill request → Print the target URL and the connection failure, exit 1, and do not retry.
- `verify-audit --from` names an index outside the ledger → Exit 2 without reporting a verdict on the chain.
- Ledger file is truncated mid-block → Exit 1 naming the first block that could not be read.
