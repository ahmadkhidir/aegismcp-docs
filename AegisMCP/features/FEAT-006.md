## FEAT-006: Human-in-the-Loop (HITL) Out-of-Band Approval Gateway

**Priority:** P0
**Effort:** M
**Depends On:** FEAT-001, FEAT-002

**User Story:**
As Elena Rostova, I want high-impact tool invocations (such as database drops, financial refunds, or code deployments) to be held in suspense until approved via Slack, email, or webhook, so that critical actions always have verified human oversight.

**Description:**
A suspended-execution gateway for dangerous or irreversible tools. When an agent calls a tool designated as `require_approval: true` in policy, AegisMCP suspends the JSON-RPC request without disconnecting the agent, dispatches an approval notification to configured endpoints (Slack interactive button, webhook, or web console), and resumes or rejects execution upon human response.

Each ticket is named by an opaque, unguessable token rendered as `appr_` followed by 26 base32 characters, reaches its 300-second time-to-live from creation, and resolves exactly once. The token is the value carried by every approval control: a Slack button, a console action, or a webhook callback.

Slack approval messages carry an Approve and a Reject button whose action identifiers are `aegis_approve` and `aegis_reject` and whose value is the ticket token. Every click lands on `POST /api/v1/integrations/slack/interactions` as an `application/x-www-form-urlencoded` body whose `payload` field holds the interaction JSON. The gateway authenticates the callback with the workspace signing secret: it recomputes `v0=` followed by the HMAC-SHA256 of `v0:<X-Slack-Request-Timestamp>:<raw body>` and compares it in constant time against `X-Slack-Signature`, and it refuses a timestamp more than 300 seconds from its own clock. An unauthenticated callback is answered `401` and changes nothing; in a deployment with no Slack workspace configured the endpoint answers `404`.

An authenticated callback is resolved through the same approval path as the console. The acting Slack user is mapped to an operator session, that operator's identity is sealed as the resolving approver, and a mapped operator without the `operator` or `admin` role is refused. The endpoint answers `200` for every callback it has authenticated, because Slack redelivers any other status: a decision that was applied is reported back with `{"replace_original": true, "text": "<decision> by <operator> at <timestamp>"}` so the channel message shows the outcome, a refused actor receives an ephemeral message stating why, and a ticket that is already resolved, cancelled, or expired yields `200` with the outcome already recorded and no second decision.

Slack notifications are posted once through the workspace's incoming-webhook URL. The deployment configures that URL and the workspace signing secret, and no bot token and no interaction response URL, so a message the gateway has posted is only ever changed by its answer to a click on that message. An approval that reaches its 300-second time-to-live expires and fails the suspended call closed with `APPROVAL_TIMEOUT`, and a pending ticket whose agent disconnects is cancelled without resuming the call. Neither expiry nor cancellation, nor a decision taken in the console or through a webhook, edits the Slack message: it stays as posted. The interaction's sealed ledger entry and the Web Console approval history carry the authoritative outcome, and a later click on a ticket that is no longer pending reports the outcome already recorded.

**Acceptance Criteria:**
- **Given** an agent calling a sensitive tool requiring human approval
  **When** the tool call is intercepted by the gateway
  **Then** the request is suspended, a unique approval token is generated, and an interactive message with Approve and Reject buttons is posted to the designated Slack channel or webhook.
- **Given** a designated human approver reviewing the pending request
  **When** the approver clicks "Approve" within the 300-second window
  **Then** AegisMCP releases the suspended JSON-RPC call to the upstream tool server and forwards the resulting response back to the waiting agent.
- **Given** an approver clicking "Reject" or the approval window expiring without an action
  **When** the decision is processed
  **Then** the tool call is denied with a JSON-RPC execution-denied error to the agent (`APPROVAL_DENIED` for a rejection, `APPROVAL_TIMEOUT` for an expiry), the outcome is sealed in the audit ledger, the console shows the ticket as rejected or expired, and the Slack message remains as posted.
- **Given** a pending ticket and a Slack operator bound to `operator` or `admin`
  **When** the operator clicks Approve
  **Then** the callback answers `200` with `replace_original` set, the ticket becomes approved, the agent's suspended call resumes, the ledger seals the mapped operator as the approver, and the channel message reports the operator and decision time.
- **Given** a pending ticket
  **When** the operator clicks Reject
  **Then** the callback answers `200` with `replace_original` set, the ticket records the rejection with reason `SLACK_REJECTED`, and the agent receives `-32002` with machine code `APPROVAL_DENIED`.
- **Given** a callback whose signature does not match the computed value, whose timestamp is more than 300 seconds from the gateway clock, or whose body is not a Slack interaction payload
  **When** it reaches the endpoint
  **Then** the gateway answers `401` and the ticket keeps its state.
- **Given** a callback from a Slack user with no operator mapping or without `operator` or `admin`
  **When** the gateway has authenticated it
  **Then** the callback answers `200` with an ephemeral refusal and the ticket keeps its state.
- **Given** a ticket already approved, rejected, expired, or cancelled
  **When** any button for it is clicked again, or Slack redelivers the same callback
  **Then** the endpoint answers `200` reporting the outcome already recorded and applies no second decision.
- **Given** a deployment with no Slack workspace configured
  **When** a request arrives on the Slack callback endpoint
  **Then** the gateway answers `404`.

**Edge Cases & Failure States:**
- Human approver does not respond before the ticket's time-to-live expires → The ticket expires, the suspended call fails closed with an `APPROVAL_TIMEOUT` error, and gateway resources are freed.
- Agent disconnects or cancels the request while approval is pending → Automatically cancel the pending approval ticket without resuming the call, and leave the Slack message as posted.
- Two approvers click at the same moment → The first decision is applied and sealed, and the second callback reports that decision without changing it.
- Ticket is resolved from the console or a webhook first → A later Slack click reports the recorded decision and changes nothing.
- Slack delivery of the approval message fails → The ticket stays pending and reachable from the Web Console queue until its 300-second expiry.
- Approver clicks after the agent's stream closed → The callback reports the cancelled ticket and the agent's call is not resumed.
