## FEAT-006: Human-in-the-Loop (HITL) Out-of-Band Approval Gateway

**Priority:** P0
**Effort:** M
**Depends On:** FEAT-001, FEAT-002

**User Story:**
As Elena Rostova, I want high-impact tool invocations (such as database drops, financial refunds, or code deployments) to be held in suspense until approved via Slack, email, or webhook, so that critical actions always have verified human oversight.

**Description:**
A suspended-execution gateway for dangerous or irreversible tools. When an agent calls a tool designated as `require_approval: true` in policy, AegisMCP suspends the JSON-RPC request without disconnecting the agent, dispatches an approval notification to configured endpoints (Slack interactive button, webhook, or web console), and resumes or rejects execution upon human response.

**Acceptance Criteria:**
- **Given** an agent calling a sensitive tool requiring human approval
  **When** the tool call is intercepted by the gateway
  **Then** the request is suspended, a unique approval token is generated, and an interactive message is posted to the designated Slack channel or webhook.
- **Given** a designated human approver reviewing the pending request
  **When** the approver clicks "Approve" within the configured TTL window (e.g. 5 minutes)
  **Then** AegisMCP releases the suspended JSON-RPC call to the upstream tool server and forwards the resulting response back to the waiting agent.
- **Given** an approver clicking "Reject" or the approval window expiring without an action
  **When** the decision is processed
  **Then** the tool call is rejected, returning a JSON-RPC execution denied error to the agent, and the decision is permanently recorded in the audit log.

**Edge Cases & Failure States:**
- Human approver does not respond before timeout expires → Auto-reject the request with `APPROVAL_TIMEOUT` error, freeing gateway resources.
- Agent disconnects or cancels the request while approval is pending → Automatically cancel the pending approval ticket and mark the Slack notification as expired.
