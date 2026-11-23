## FEAT-011: Automated Compliance Report Generator (EU AI Act & SOC 2)

**Priority:** P2
**Effort:** M
**Depends On:** FEAT-007

**User Story:**
As David Chen, I want to export cryptographically signed compliance attestations of agent tool boundaries and execution histories, so that enterprise customers can pass vendor security reviews without friction.

**Description:**
A compliance reporting utility that compiles operational telemetry and tamper-evident audit records into standardized compliance reports. Reports map agent permissions, tool usage statistics, security violations, and human approval interventions directly to control frameworks including EU AI Act (Article 12 & 14) and SOC 2 Type II Security & Confidentiality principles. A reporting period too large to compile inside the request is generated in the background and delivered to the deployment's notification channels as a signed archive.

**Acceptance Criteria:**
- **Given** an operator authenticated through the enterprise identity provider holding the `auditor` or `admin` role
  **When** the operator sends `POST /api/v1/compliance/export` with `framework` `EU_AI_ACT` or `SOC2_TYPE2`, `format` `JSON`, `CSV`, or `PDF`, inclusive ISO-8601 `from` and `to` bounds spanning at most 31 days and at most 100,000 sealed audit blocks, and an optional `agentId` restricting the report to one identity
  **Then** the response is HTTP 200 carrying `reportId`, `framework`, `format`, `status` `COMPLETED`, `generatedAt`, `recordCount`, `violationsCount`, `approvalsCount`, `merkleRoot`, `signature`, and `content` — the report text for `JSON` and `CSV`, the Base64 encoding of the signed PDF archive for `PDF`.
- **Given** a reporting period spanning more than 31 days or more than 100,000 sealed audit blocks
  **When** the request is accepted
  **Then** the response is HTTP 202 carrying `reportId` and `status` `GENERATING`, generation continues after the response, and the completed archive is delivered to every notification channel configured for the deployment: a webhook whose JSON body carries the same fields as the synchronous response, and each configured recipient address receiving the signed archive as an email attachment.
- **Given** a completed `EU_AI_ACT` report
  **Then** it presents the evidence for Article 12 record-keeping and Article 14 human oversight; and a completed `SOC2_TYPE2` report presents the Security and Confidentiality common criteria. Both itemize the policy definitions in force, execution counts per tool and agent, every human approval with its decision timestamp and reviewer, and a security-exception section holding each violation, blocked payload, and resolution trace inside the window.
- **Given** a delivered report of any format
  **When** a recipient verifies it
  **Then** `merkleRoot` recomputes from the sealed blocks included in the report, and `signature`, a detached signature over the report digest made with the deployment's signing key, verifies against the published public key.
- **Given** a request with an undeclared `framework` or `format`, a `from` or `to` that is not an ISO-8601 date-time, or a `from` later than `to`
  **Then** the response is HTTP 400 and no report is generated.
- **Given** a request whose bearer token is missing, expired, or invalid
  **Then** the response is HTTP 401.
- **Given** a verified operator holding none of `auditor` or `admin`
  **Then** the response is HTTP 403 and no report is generated.
- **Given** a hash-chain verification failure inside the requested range
  **When** generation runs
  **Then** no report is produced or delivered, the response is HTTP 409 carrying `brokenBlockIndex` for the first compromised block, and an integrity alert naming the range and block reaches the configured compliance notification channel.
- **Given** a completed generation, synchronous or background
  **Then** `EVT-012` fires with the framework, the record count, and the signature verification status.

**Edge Cases & Failure States:**
- A reporting period containing no audit blocks → a completed report with `recordCount` 0 and empty sections, not an error.
- The signing key is unavailable at generation time → generation aborts, the failure is recorded in the ledger, and the compliance notification channel is alerted; an unsigned report is never delivered.
- A webhook or email delivery is rejected → the gateway retries with exponential backoff for up to 15 minutes, then records the failed delivery in the ledger and alerts the compliance notification channel.
