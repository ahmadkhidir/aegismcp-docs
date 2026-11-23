## FEAT-010: Emergency Global Kill-Switch & Circuit Breaker

**Priority:** P1
**Effort:** S
**Depends On:** FEAT-001

**User Story:**
As Marcus Vance, I want an instant kill-switch API and CLI command to immediately quarantine rogue agents or sever all connections to compromised MCP servers, so that runaway agent loops can be arrested within seconds.

**Description:**
A rapid-response emergency control mechanism that allows administrators to sever active connections and block new invocations at multiple granularities: single agent session, specific agent identity, single MCP server, or a global emergency lockdown. It also incorporates an automated circuit breaker that trips when an agent exceeds safe invocation velocity thresholds. Every action is recorded in the tamper-evident ledger with the operator, target, and reason, and stays in force until an authorized operator releases it.

**Acceptance Criteria:**
- **Given** an operator authenticated through the enterprise identity provider holding the `operator` or `admin` role
  **When** the operator sends `POST /api/v1/emergency/kill` with `targetType` `agent` or `session` and the matching `targetId`
  **Then** every active transport stream for that target is closed before the response is returned, a quarantined agent identity additionally has later connection attempts refused, and the response is HTTP 200 carrying `status` `QUARANTINED`, `affectedSessions` with the number of streams closed, and an ISO-8601 `timestamp`.
- **Given** a compromised upstream MCP server identified by an operator holding the `operator` or `admin` role
  **When** the operator sends the same request with `targetType` `server` and that server's `targetId`
  **Then** every route to that server is severed across all agent sessions and the response carries `status` `SEVERED`.
- **Given** an operator requesting a global lockdown with `targetType` `global`, no `targetId`, and a multi-factor re-confirmation of that operator completed within the preceding five minutes
  **When** the request is accepted
  **Then** all new tool invocations and connection attempts are refused cluster-wide and the response carries `status` `LOCKDOWN`.
- **Given** a global lockdown request whose multi-factor re-confirmation is absent or older than five minutes
  **Then** the response is HTTP 403 with no quarantine or lockdown applied.
- **Given** a request whose `targetType` is not one of `session`, `agent`, `server`, `global`, whose `targetId` is missing for a scoped target, or which supplies a `targetId` with `global`
  **Then** the response is HTTP 400 with no quarantine or lockdown applied.
- **Given** a request naming a session, agent identity, or upstream server that does not exist
  **Then** the response is HTTP 404 with no quarantine or lockdown applied.
- **Given** a request carrying no access token, an expired token, or an invalid token
  **Then** the response is HTTP 401.
- **Given** a verified operator holding none of `operator` or `admin`
  **Then** the response is HTTP 403 with no quarantine or lockdown applied.
- **Given** a multi-node gateway cluster and an accepted kill action
  **When** the signal is distributed to peer nodes
  **Then** every peer applies it within 500 milliseconds; when a peer cannot be reached inside that budget the response is HTTP 503, the nodes that applied it keep it in force, and the operator retries.
- **Given** an agent that issues more than 100 tool invocations inside a rolling 10-second window
  **When** the threshold is breached
  **Then** the circuit breaker closes the agent's transport streams and refuses that identity until an operator releases it, and `EVT-011` records the breach with its scope and target.
- **Given** an active quarantine or lockdown
  **When** an operator holding `operator` or `admin` sends `POST /api/v1/emergency/release` with the matching `targetType`, the scoped `targetId`, and a `reason`
  **Then** the target may connect and invoke tools again, the release is appended to the ledger with the operator identity and reason, and the response is HTTP 200 carrying `status` `RELEASED` for a scoped target or `LOCKDOWN_LIFTED` for `global`, with an ISO-8601 `timestamp`. A `global` release requires the same five-minute multi-factor re-confirmation and returns HTTP 403 without it.
- **Given** a release request whose `targetId` carries no active quarantine or lockdown
  **Then** the response is HTTP 404 with the target's state unchanged; a release request for a scoped target without a `reason` returns HTTP 400.
- **Given** an operator running `aegis-cli kill --agent <agent-id>`
  **When** the gateway accepts the request
  **Then** the CLI prints the returned status, affected session count, and timestamp and exits 0, and every documented HTTP failure exits non-zero with its status and reason on stderr.

**Edge Cases & Failure States:**
- Administrator accidentally trips the global kill-switch → the CLI refuses a `global` target without `--confirm-lockdown`, and the gateway independently refuses it without a fresh multi-factor re-confirmation.
- A gateway node restarts while quarantines or a lockdown are in force → the state is restored before the node accepts traffic, so a restart cannot lift an emergency control.
- A kill action arrives while the target has a tool invocation in flight → the invocation is terminated with a `-32001 Policy Violation` error rather than completed, and the stream closes behind it.
