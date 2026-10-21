## Screen 5: Audit Explorer

**Route:** `/audit`
**Platform:** Web
**Presentation:** Page
**Purpose:** Provide a forensic search, verification, and compliance reporting interface over the immutable, cryptographically hash-chained audit ledger.
**Visual Focus:** Cryptographic Ledger Integrity status banner atop a dense, searchable event record table.
**Action Hierarchy:** Primary: Export Compliance Report; Secondary: Run Cryptographic Chain Verification; Tertiary: Filter Records.
**Entry Points:** [Security Dashboard](01-dashboard.md), [Approvals Queue](03-approvals-queue.md), or left navigation rail.
**Exit Points:** [Security Dashboard](01-dashboard.md), [Live Traffic Inspector](02-live-inspector.md).

### Layout

**Web layout:**
- Ledger Integrity Banner: Displays overall cryptographic chain status (e.g. "Cryptographic Chain Verified — 1,248,390 sealed blocks with 0 discrepancies"), last verification timestamp, and "Verify Now" button.
- Query and Filter Toolbar: Date & Time range picker, Search input (for Agent ID, Tool Name, or SHA-256 Block Hash), Decision filter (Allowed / Blocked / Approved), and "Export Report" button.
- Forensic Event Data Table: Columns for Timestamp (UTC), Block Index, Cryptographic Hash (truncated with copy button), Agent Identity, Tool Name, Policy Decision, Human Approver (if applicable), and "View Proof" action.
- Pagination / Infinite Scroll controls with total matching record count.

### Components

| Component | Type | Platform | Purpose | Data Source |
|---|---|---|---|---|
| Integrity banner | Panel | Web | Visualizes SHA-256 ledger validation status | GET `/api/v1/audit/integrity` |
| Filter toolbar | Toolbar | Web | Filters historical audit records by date, agent, and hash | Client query state |
| Audit record table | Table | Web | Displays searchable immutable ledger entries | GET `/api/v1/audit/records` |
| Hash proof drawer | Panel | Web | Inspects SHA-256 block hash, parent hash, and digital signature | Selected block record |
| Export dialog | Modal | Web | Configures compliance report format (EU AI Act, SOC 2) | User selection |

### Overlays

| Overlay | Pattern | Trigger | Behavior | States |
|---|---|---|---|---|
| Compliance Export Modal | Dialog | "Export Compliance Report" button | Modal dialog allowing date range and framework selection (EU AI Act Article 12/14 or SOC 2 Type II) to generate a signed PDF/JSON archive | Default, Generating Hashes, Ready for Download, Error |

### Data Requirements

**Data Needed:**
- Ledger integrity metrics: total verified blocks, chain valid boolean, last verified timestamp.
- Paginated audit log records: block index, block hash, previous block hash, timestamp, agent identity, tool called, parameter hash, decision, human approver identity, latency.

**Data Flow:**
1. Screen mounts → fetch integrity status (`GET /api/v1/audit/integrity`) and initial page of records (`GET /api/v1/audit/records?limit=50`).
2. User submits search query → execute filtered search against audit database.
3. User clicks "Verify Now" → trigger asynchronous ledger recalculation; display real-time progress bar.
4. User clicks "Export Compliance Report" → open modal; on confirm, generate signed compliance bundle.

### States

| State | Condition | What the User Sees |
|---|---|---|
| **Loading** | Records query in progress | Skeleton table rows with shimmering animation |
| **Loaded** | Audit records retrieved | Populated table with monospace hash identifiers and status pills |
| **Verifying** | Manual chain verification running | Progress bar inside integrity banner showing checked block count |
| **Integrity Compromise** | Cryptographic hash mismatch detected | High-severity crimson alert banner identifying damaged block index |
| **Empty Search** | Query returns zero matching records | "No audit records found matching the specified query criteria" |

**Edge Cases:**
- Exporting records across multi-gigabyte date ranges → Asynchronously generate archive in background and display download notification toast when ready.
- Auditor inputs raw SHA-256 hash in search bar → Instantly navigate to exact block index and display cryptographic parent-child link visualization.
