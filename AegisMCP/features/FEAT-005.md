## FEAT-005: Outbound Tool Response Secret & PII Scrubber

**Priority:** P0
**Effort:** M
**Depends On:** FEAT-001

**User Story:**
As Elena Rostova, I want AegisMCP to automatically redact API keys, private credentials, and sensitive personal data from tool outputs before they return to the agent, so that confidential information never leaks into model context windows.

**Description:**
An inline data loss prevention (DLP) scanner that intercepts upstream tool execution results before delivering them back to the agent client. It applies high-performance pattern matching (Aho-Corasick and optimized regex algorithms) to detect and redact API keys (AWS, GitHub, OpenAI tokens), private cryptographic keys, and personal identifiers (SSNs, credit card numbers, email addresses).

**Acceptance Criteria:**
- **Given** an upstream MCP tool returning text content containing an AWS secret key or GitHub Personal Access Token
  **When** the response traverses the outbound inspection pipeline
  **Then** the credential string is replaced with a standard redaction marker (e.g. `[REDACTED_SECRET:AWS_KEY]`) before reaching the agent.
- **Given** a tool output containing personal identifiable information (PII) such as credit card numbers or Social Security Numbers
  **When** evaluated against active PII masking rules
  **Then** the sensitive values are masked (e.g. `[REDACTED_PII:CREDIT_CARD]`) and a redaction telemetry event is recorded.
- **Given** a clean tool output free of matching patterns
  **When** passed through the scrubber
  **Then** the payload streams directly to the agent without modification or latency penalty.

**Edge Cases & Failure States:**
- Huge tool output payload (e.g. 50MB file read or database dump) → Stream chunks through sliding buffer without loading entire payload into heap memory; enforce configurable max response size limit.
- Redaction scanner encounters timeout on deeply nested JSON structures → Fail-safe by masking ambiguous fields and logging a scanner warning.
