# Data Model & Schema

## Overview

AegisMCP exchanges and persists structured data across its high-performance proxy pipeline, in-memory policy cache, and append-only cryptographic audit ledger. The data model is relational and referential: agent identities map to declarative security policies, intercepted tool invocations link to approval tickets, and all operational events are recorded into an immutable, SHA-256 hash-chained ledger. Entities are formally defined below using JSON Schema (Draft 2020-12).

---

## Entities

### AgentIdentity
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aegismcp.dev/schemas/agent-identity.schema.json",
  "title": "AgentIdentity",
  "description": "An authenticated autonomous AI agent workload registered with the security gateway.",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "description": "Unique stable identifier for the agent (e.g. SPIFFE ID 'spiffe://cluster.local/ns/agents/sa/support-bot' or UUID)."
    },
    "name": {
      "type": "string",
      "description": "Human-readable label for the agent workload."
    },
    "authType": {
      "type": "string",
      "enum": ["spiffe_svid", "oidc_jwt", "api_key"],
      "description": "Cryptographic authentication mechanism used by the agent."
    },
    "status": {
      "type": "string",
      "enum": ["active", "quarantined", "revoked"],
      "description": "Current lifecycle and operational state of the agent."
    },
    "assignedPolicyId": {
      "type": "string",
      "description": "References the SecurityPolicy assigned to govern this agent."
    },
    "createdAt": {
      "type": "string",
      "format": "date-time",
      "description": "Timestamp when the agent identity was first registered (UTC)."
    }
  },
  "required": ["id", "name", "authType", "status", "assignedPolicyId", "createdAt"]
}
```

### SecurityPolicy
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aegismcp.dev/schemas/security-policy.schema.json",
  "title": "SecurityPolicy",
  "description": "A declarative Policy-as-Code document defining tool access rules, argument constraints, and approval requirements.",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "description": "Stable unique identifier for the policy (e.g. 'pol_staging_db_v1')."
    },
    "name": {
      "type": "string",
      "description": "Descriptive title of the security policy."
    },
    "version": {
      "type": "string",
      "description": "Semantic version string of the policy configuration."
    },
    "rawYaml": {
      "type": "string",
      "description": "Complete declarative YAML policy specification text."
    },
    "status": {
      "type": "string",
      "enum": ["active", "draft", "archived"],
      "description": "Deployment status of the policy version."
    },
    "gitCommitHash": {
      "type": "string",
      "description": "Git commit SHA tracking policy origin in GitOps repositories."
    },
    "createdAt": {
      "type": "string",
      "format": "date-time",
      "description": "Timestamp when this policy version was created (UTC)."
    }
  },
  "required": ["id", "name", "version", "rawYaml", "status", "createdAt"]
}
```

### ApprovalTicket
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aegismcp.dev/schemas/approval-ticket.schema.json",
  "title": "ApprovalTicket",
  "description": "A suspended tool invocation awaiting Human-in-the-Loop review and authorization.",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "description": "Unique identifier for the approval ticket (e.g. 'appr_8f91a2')."
    },
    "agentId": {
      "type": "string",
      "description": "References the AgentIdentity requesting the tool execution."
    },
    "toolName": {
      "type": "string",
      "description": "Target MCP tool name invoked by the agent."
    },
    "argumentsPayload": {
      "type": "string",
      "description": "JSON-serialized input arguments passed to the tool."
    },
    "status": {
      "type": "string",
      "enum": ["pending", "approved", "rejected", "expired"],
      "description": "Current authorization state of the ticket."
    },
    "triggerPolicyId": {
      "type": "string",
      "description": "References the SecurityPolicy rule that mandated human approval."
    },
    "reviewedBy": {
      "type": "string",
      "description": "Identifier of the human reviewer who authorized or rejected the ticket (personal data)."
    },
    "rejectionReason": {
      "type": "string",
      "description": "Contextual reason provided by the human reviewer on rejection."
    },
    "expiresAt": {
      "type": "string",
      "format": "date-time",
      "description": "Timestamp when the ticket automatically expires and fails closed (UTC)."
    },
    "createdAt": {
      "type": "string",
      "format": "date-time",
      "description": "Timestamp when the approval ticket was created (UTC)."
    }
  },
  "required": ["id", "agentId", "toolName", "argumentsPayload", "status", "triggerPolicyId", "expiresAt", "createdAt"]
}
```

### AuditBlock
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aegismcp.dev/schemas/audit-block.schema.json",
  "title": "AuditBlock",
  "description": "An immutable, cryptographically hash-chained record sealing one intercepted tool interaction.",
  "type": "object",
  "properties": {
    "blockIndex": {
      "type": "integer",
      "minimum": 0,
      "description": "Sequential monotonically increasing 64-bit integer block number."
    },
    "blockHash": {
      "type": "string",
      "description": "SHA-256 cryptographic hash of this block's contents and signatures."
    },
    "parentHash": {
      "type": "string",
      "description": "SHA-256 hash of the immediately preceding block in the chain."
    },
    "agentId": {
      "type": "string",
      "description": "References the AgentIdentity that initiated the interaction."
    },
    "toolName": {
      "type": "string",
      "description": "Name of the MCP tool invoked."
    },
    "decision": {
      "type": "string",
      "enum": ["allow", "block", "require_approval"],
      "description": "Gateway security decision applied to the interaction."
    },
    "redactionCount": {
      "type": "integer",
      "minimum": 0,
      "description": "Count of credential or PII instances scrubbed from the response by DLP."
    },
    "latencyMs": {
      "type": "number",
      "minimum": 0.0,
      "description": "Total proxy processing latency in milliseconds."
    },
    "timestamp": {
      "type": "string",
      "format": "date-time",
      "description": "Immutable timestamp when the block was sealed (UTC)."
    }
  },
  "required": ["blockIndex", "blockHash", "parentHash", "agentId", "toolName", "decision", "redactionCount", "latencyMs", "timestamp"]
}
```

### SecurityViolation
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aegismcp.dev/schemas/security-violation.schema.json",
  "title": "SecurityViolation",
  "description": "A security incident generated when an agent violates policy, executes a prohibited sequence, or triggers injection heuristics.",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "description": "Unique identifier for the security violation event."
    },
    "agentId": {
      "type": "string",
      "description": "References the AgentIdentity responsible for the violation."
    },
    "violationType": {
      "type": "string",
      "enum": [
        "prompt_injection",
        "tool_chaining_anomaly",
        "schema_violation",
        "path_traversal",
        "unauthorized_tool",
        "rate_limit_exceeded"
      ],
      "description": "Classification category of the detected security violation."
    },
    "blockHash": {
      "type": "string",
      "description": "References the AuditBlock sealing this violation event."
    },
    "severity": {
      "type": "string",
      "enum": ["low", "medium", "high", "critical"],
      "description": "Assessed risk severity level of the incident."
    },
    "details": {
      "type": "string",
      "description": "Diagnostic explanation and parameter snippet of the blocked payload."
    },
    "createdAt": {
      "type": "string",
      "format": "date-time",
      "description": "Timestamp when the violation occurred (UTC)."
    }
  },
  "required": ["id", "agentId", "violationType", "blockHash", "severity", "details", "createdAt"]
}
```

### ConsoleOperatorSession
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aegismcp.dev/schemas/console-operator-session.schema.json",
  "title": "ConsoleOperatorSession",
  "description": "An authenticated human operator session established via enterprise SSO/OIDC with active RBAC claims.",
  "type": "object",
  "properties": {
    "sub": {
      "type": "string",
      "description": "Cryptographic subject identifier from the enterprise IdP or SPIFFE user namespace."
    },
    "name": {
      "type": "string",
      "description": "Full display name of the human operator."
    },
    "email": {
      "type": "string",
      "format": "email",
      "description": "Corporate email address associated with the enterprise SSO identity."
    },
    "roles": {
      "type": "array",
      "items": {
        "type": "string",
        "enum": ["admin", "operator", "auditor", "viewer"]
      },
      "description": "RBAC roles assigned by enterprise IdP group claims."
    },
    "title": {
      "type": "string",
      "description": "Operational organizational title (e.g. SecOps Admin, Platform Engineer, CISO / Auditor)."
    },
    "idp": {
      "type": "string",
      "description": "Authority issuer identifier of the authenticating Identity Provider."
    },
    "sessionExpiresAt": {
      "type": "integer",
      "description": "Unix timestamp in milliseconds when the active 15-minute inactivity session window expires."
    }
  },
  "required": ["sub", "name", "email", "roles", "idp", "sessionExpiresAt"]
}
```

---

## Relationships

- **AgentIdentity** `assignedPolicyId` references **SecurityPolicy** — each agent is bound to exactly one active security policy configuration.
- **ApprovalTicket** `agentId` references **AgentIdentity** — each approval request originates from exactly one authenticated agent.
- **ApprovalTicket** `triggerPolicyId` references **SecurityPolicy** — each approval requirement is mandated by a rule in a specific policy.
- **AuditBlock** `agentId` references **AgentIdentity** — each immutable ledger entry records an interaction executed by one agent.
- **SecurityViolation** `agentId` references **AgentIdentity** — each security violation event is attributed to one agent identity.
- **SecurityViolation** `blockHash` references **AuditBlock** — each security violation is sealed within an immutable audit block.
- **ConsoleOperatorSession** `roles` govern administrative execution permissions across **SecurityPolicy**, **ApprovalTicket**, and **AuditBlock** actions.

---

## Personal Data & Compliance Tagging

In accordance with data governance obligations outlined in [nfr](nfr.md):
- **ConsoleOperatorSession** `name` and `email`: Encapsulate authenticated human operator identity, governed by enterprise IdP directory retention and access policies.
- **ApprovalTicket** `reviewedBy`: Contains human reviewer email or username. Subject to statutory access and deletion procedures.
- **AuditBlock** and **SecurityViolation** records: Persisted in the tamper-evident ledger for 365 days under regulatory retention mandates ([NFR-041](nfr.md)), and archived in customer private storage.
