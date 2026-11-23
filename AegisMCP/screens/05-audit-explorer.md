## Screen 5: Audit Explorer

**Route:** `/audit`
**Device Classes:** All
**Purpose:** Provide a forensic search, verification, and compliance reporting interface over the immutable, cryptographically hash-chained audit ledger.
**Visual Focus:** Cryptographic Ledger Integrity status banner atop a dense, searchable event record table.
**Entry Points:** [Security Dashboard](01-dashboard.md), [Approvals Queue](03-approvals-queue.md), or left navigation rail.
**Exit Points:** [Security Dashboard](01-dashboard.md), [Live Traffic Inspector](02-live-inspector.md).

### Layout

**Web layout:**
- Ledger Integrity Banner: Displays the result of the most recent completed cryptographic chain verification (e.g. "Cryptographic Chain Verified — 1,248,390 sealed blocks with 0 discrepancies"), its completion timestamp, and a "Refresh Status" button that re-reads the reported result. Full chain recalculation runs on the deployment's verification schedule, not from the console.
- Query and Filter Toolbar: Date & Time range picker, Search input (for Agent ID, Tool Name, or SHA-256 Block Hash), Decision filter (Allowed / Blocked / Approved), and "Export Report" button.
- Forensic Event Data Table: Columns for Timestamp (UTC), Block Index, Cryptographic Hash (truncated with copy button), Agent Identity, Tool Name, Arguments Hash, Policy Decision, Human Approver (if applicable), and "View Proof" action.
- Pagination / Infinite Scroll controls with total matching record count.

### Components

| Component | Type | Purpose | Available On | Priority | Data Source |
|---|---|---|---|---|---|
| Integrity banner | Panel | Shows the last completed SHA-256 ledger validation and its timestamp | Mobile, Tablet, Desktop | Secondary | READ[Screen state(ledgerIntegrity)] |
| Filter toolbar | Toolbar | Filters historical audit records by date, agent, and hash | Mobile, Tablet, Desktop | Tertiary | WRITE[Screen state(activeQuery)] |
| Audit record table | Table | Displays searchable immutable ledger entries | Mobile, Tablet, Desktop | — | READ[AuditBlock(blockIndex, blockHash, timestamp, agentId, toolName, argumentsHash, decision, reviewedBy)] |
| Hash proof drawer | Panel | Inspects SHA-256 block hash, parent hash, arguments hash, and digital signature | Mobile, Tablet, Desktop | — | READ[AuditBlock(blockIndex, blockHash, parentHash, argumentsHash)] |
| Export dialog | Modal | Configures compliance report format (EU AI Act, SOC 2) and reports generation and delivery outcome | Mobile, Tablet, Desktop | Primary | WRITE[Screen state(exportRequest)] |

### Overlays

| Overlay | Pattern | Trigger | Behavior | States |
|---|---|---|---|---|
| Compliance Export Modal | Dialog | "Export Compliance Report" button | Selects date range and framework (EU AI Act Article 12/14 or SOC 2 Type II) and submits the export. A range inside the synchronous ceiling returns a signed report for download; a larger range is accepted for background generation and the signed archive is delivered to the deployment's notification channels, retrying a rejected delivery for up to 15 minutes before recording the failure and alerting the compliance notification channel. The console shows the accepted request, not the internal retry sequence. | Default, Generating, Ready for Download, Accepted for Delivery, Error |

### Data Requirements

**Data Needed:**
- Ledger integrity metrics: total verified blocks, chain valid boolean, last verified timestamp.
- Paginated audit log records: block index, block hash, previous block hash, timestamp, agent identity, tool called, arguments hash, decision, human approver identity (empty when no approval applied), redaction count, latency.

**Data Flow:**
1. Screen mounts → fetch integrity status (`GET /api/v1/audit/integrity`) and initial page of records (`GET /api/v1/audit/records?limit=50`).
2. User submits search query → execute filtered search against audit database.
3. User clicks "Refresh Status" → re-read the last completed verification result and its timestamp.
4. User clicks "Export Compliance Report" → open modal; on confirm, submit the export. A synchronous completion offers the signed archive for download; an accepted background request is announced with the delivery channels that will carry the archive.

### States

| Scope | State | Condition | What the User Sees |
|---|---|---|---|
| Screen | **Loading** | Records query in progress | Skeleton table rows with shimmering animation |
| Screen | **Loaded** | Audit records retrieved | Populated table with monospace hash identifiers and status pills |
| Integrity banner | **Integrity Unavailable** | Ledger store unreachable | Warning banner showing the last known verification result and its time, with a "Retry Connection" action |
| Integrity banner | **Integrity Compromise** | Cryptographic hash mismatch detected | High-severity crimson alert banner identifying damaged block index |
| Audit record table | **Empty Search** | Query returns zero matching records | "No audit records found matching the specified query criteria" |

**Edge Cases:**
- Exporting records across multi-gigabyte date ranges → Accept the request for background generation, show the accepted state, and deliver the signed archive through the deployment's notification channels; the console does not poll generation or delivery progress, and a delivery that exhausts its retry window is reported to the compliance notification channel.
- Auditor inputs raw SHA-256 hash in search bar → Instantly navigate to exact block index and display cryptographic parent-child link visualization.
