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
    "agentIntent": {
      "type": ["string", "null"],
      "description": "Agent-supplied justification for the invocation, presented to the reviewer as the intent description; null when the call carried none."
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
    "processedAt": {
      "type": ["string", "null"],
      "format": "date-time",
      "description": "Timestamp when a human decision resolved the ticket, used to order approval history; null while the ticket is pending."
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
  "required": ["id", "agentId", "toolName", "argumentsPayload", "agentIntent", "status", "triggerPolicyId", "expiresAt", "createdAt", "processedAt"]
}
```

### AdministrativeAuditEntry
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aegismcp.dev/schemas/administrative-audit-entry.schema.json",
  "title": "AdministrativeAuditEntry",
  "description": "An immutable, cryptographically hash-chained record sealing one administrative action, its target, and its outcome.",
  "type": "object",
  "properties": {
    "blockIndex": {
      "type": "integer",
      "minimum": 0,
      "description": "Position of this entry in the shared ledger chain, drawn from the same monotonically increasing 64-bit sequence as every tool-interaction block."
    },
    "blockHash": {
      "type": "string",
      "description": "SHA-256 cryptographic hash of this entry's contents and signatures."
    },
    "parentHash": {
      "type": "string",
      "description": "SHA-256 hash of the immediately preceding sealed entry in the chain, whether it records a tool interaction or an administrative action."
    },
    "actor": {
      "type": "string",
      "description": "Verified principal that performed the action: the operator's authenticated subject for a console or command-line action, the CI/CD service account for a pipeline action, or the Git commit that introduced the document for a GitOps reload (personal data)."
    },
    "secondApprover": {
      "type": ["string", "null"],
      "description": "Distinct verified operator whose authorization a dual-operator action required and received; null when the action required one operator (personal data)."
    },
    "action": {
      "type": "string",
      "enum": ["policy_publish", "policy_activation", "policy_reload", "approval_decision", "kill", "release", "role_change"],
      "description": "Administrative action the entry seals."
    },
    "target": {
      "type": "string",
      "description": "Object the action applied to: the policy name and version for a publish, activation, or reload; the approval ticket identifier for an approval decision; the emergency target type and identifier, or the global scope, for a kill or release; the operator subject whose role assignment changed for a role change."
    },
    "outcome": {
      "type": "string",
      "enum": ["accepted", "rejected", "refused"],
      "description": "Result of the action: accepted when it completed and its effect is in force, rejected when it was denied and no state changed, refused when the deployment's configuration does not permit the action at all."
    },
    "reasonCategory": {
      "type": ["string", "null"],
      "description": "Identifier from the deployment's audit taxonomy naming why the action was taken, refused, or rejected, such as a base-version conflict, an invalid document, or an emergency declaration; null when the action carried none."
    },
    "reason": {
      "type": ["string", "null"],
      "description": "Operator-supplied justification sealed with the action, as required for a release; null when the action carried none (personal data)."
    },
    "gitCommitHash": {
      "type": ["string", "null"],
      "description": "Commit whose content produced the action when the action originated from a Git commit, as on a GitOps reload or a pipeline-driven publish or activation; on a reload this is the same commit the actor names, and it is null otherwise."
    },
    "timestamp": {
      "type": "string",
      "format": "date-time",
      "description": "Immutable timestamp when the entry was sealed (UTC)."
    }
  },
  "required": ["blockIndex", "blockHash", "parentHash", "actor", "secondApprover", "action", "target", "outcome", "reasonCategory", "reason", "gitCommitHash", "timestamp"]
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
      "description": "Position of this block in the shared ledger chain, drawn from the same monotonically increasing 64-bit sequence as every administrative audit entry."
    },
    "blockHash": {
      "type": "string",
      "description": "SHA-256 cryptographic hash of this block's contents and signatures."
    },
    "parentHash": {
      "type": "string",
      "description": "SHA-256 hash of the immediately preceding sealed entry in the chain, whether it records a tool interaction or an administrative action."
    },
    "agentId": {
      "type": "string",
      "description": "References the AgentIdentity that initiated the interaction."
    },
    "toolName": {
      "type": "string",
      "description": "Name of the MCP tool invoked."
    },
    "argumentsHash": {
      "type": "string",
      "description": "SHA-256 digest of the canonicalized request arguments sealed with the decision, so a recorded invocation can be matched to its parameter payload without storing or exporting the arguments."
    },
    "decision": {
      "type": "string",
      "enum": ["allow", "block", "require_approval"],
      "description": "Gateway security decision applied to the interaction."
    },
    "reviewedBy": {
      "type": ["string", "null"],
      "description": "Verified operator identity that resolved the approval for this interaction; null when no approval applied (personal data)."
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
  "required": ["blockIndex", "blockHash", "parentHash", "agentId", "toolName", "argumentsHash", "decision", "reviewedBy", "redactionCount", "latencyMs", "timestamp"]
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
- **AdministrativeAuditEntry** `actor` names the **ConsoleOperatorSession** subject that performed an administrative action, or the Git commit that introduced the document on a GitOps reload, so each action and its principal are sealed in the same record.
- **AdministrativeAuditEntry** `secondApprover` names the second **ConsoleOperatorSession** subject whose authorization a dual-operator action required.
- **AdministrativeAuditEntry** and **AuditBlock** form one continuous chain: each `blockIndex` is drawn from a single monotonically increasing sequence and each `parentHash` names the `blockHash` of the immediately preceding sealed entry of either kind, so one verification covers tool interactions and administrative actions together.
- **AdministrativeAuditEntry** `target` names the **SecurityPolicy** name and version for a policy action, and `gitCommitHash` names the commit that introduced that version.
- **AdministrativeAuditEntry** with action `approval_decision` deliberately seals the human decision against the **ApprovalTicket** identifier while the **AuditBlock** for the same ticket seals the interaction and its `reviewedBy` outcome, so the decision and the invocation stay separately searchable in one chain.
- **AuditBlock** `agentId` references **AgentIdentity** — each immutable ledger entry records an interaction executed by one agent.
- **AuditBlock** `reviewedBy` names the **ConsoleOperatorSession** subject that resolved a required approval, so the decision and its approver are sealed in the same record.
- **SecurityViolation** `agentId` references **AgentIdentity** — each security violation event is attributed to one agent identity.
- **SecurityViolation** `blockHash` references **AuditBlock** — each security violation is sealed within an immutable audit block.
- **ConsoleOperatorSession** `roles` govern administrative execution permissions across **SecurityPolicy**, **ApprovalTicket**, **AuditBlock**, and **AdministrativeAuditEntry** actions.

---

## Personal Data & Compliance Tagging

In accordance with data governance obligations outlined in [nfr](nfr.md):
- **ConsoleOperatorSession** `name` and `email`: Encapsulate authenticated human operator identity, governed by enterprise IdP directory retention and access policies.
- **ApprovalTicket** `reviewedBy`: Contains human reviewer email or username. Subject to statutory access and deletion procedures.
- **AdministrativeAuditEntry** `actor`, `secondApprover`, and `reason`: Contain the authenticated operator identity and the operator-supplied justification; governed by the same access and deletion procedures as the approving reviewer identity.
- **AuditBlock** `reviewedBy`: Repeats the resolving reviewer identity inside the sealed record so the approval decision travels with the ledger entry; governed by the same access and deletion procedures.
- **AuditBlock** `argumentsHash`: Holds only a digest of the request arguments, so parameter material itself is never exposed through audit queries or exports.
- **AuditBlock**, **AdministrativeAuditEntry**, and **SecurityViolation** records: Persisted in the tamper-evident ledger for 365 days under regulatory retention mandates ([NFR-041](nfr.md)), and archived in customer private storage.
