## Screen 4: Policy Manager

**Route:** `/policies`
**Platform:** Web
**Presentation:** Page
**Purpose:** Allow security and platform engineers to author, inspect, simulate, and deploy declarative Policy-as-Code rules governing agent access to MCP tools.
**Visual Focus:** Split-pane interface featuring a YAML policy editor alongside an interactive Rule Test Simulator.
**Action Hierarchy:** Primary: Deploy Changes (or "Create PR" in GitOps mode); Secondary: Test Policy against Sample Invocations; Tertiary: Discard Draft.
**Entry Points:** [Security Dashboard](01-dashboard.md), [Live Traffic Inspector](02-live-inspector.md).
**Exit Points:** [Security Dashboard](01-dashboard.md), [Live Traffic Inspector](02-live-inspector.md).

### Layout

**Web layout:**
- Top action bar displaying Active Policy Version, Git commit hash, synchronization status (Synced / Draft Modified), and "Deploy Policy" button.
- Policy selector sidebar (left 25%): List of loaded policy files, agent group bindings, and tool whitelists.
- Center-right split workspace (75%):
  - Left pane: Monaco-based YAML policy editor with syntax validation, rule schema auto-complete, and inline error highlighting.
  - Right pane: Policy Simulator & Impact Analyzer: allows entering a sample agent identity, tool name, and arguments to immediately test whether it would be Allowed, Blocked, or routed to Human Approval.

### Components

| Component | Type | Platform | Purpose | Data Source |
|---|---|---|---|---|
| Version bar | Toolbar | Web | Displays current active policy version and GitOps sync state | GET `/api/v1/policies/active` |
| Policy list | List | Web | Selects between active policies, draft versions, and templates | GET `/api/v1/policies` |
| Code editor | Panel | Web | Edits YAML policy rules with schema validation and linting | Local editor state |
| Simulator panel | Card | Web | Interactive form to test rule evaluation against mock payloads | POST `/api/v1/policies/simulate` |
| Diff viewer | Dialog | Web | Visualizes differences between active policy and current draft | Client-side diff |

### Overlays

| Overlay | Pattern | Trigger | Behavior | States |
|---|---|---|---|---|
| Deploy Confirmation Modal | Dialog | "Deploy Policy" button | Shows policy diff summary and requires explicit confirmation before pushing changes | Default, Validating, Submitting |

### Data Requirements

**Data Needed:**
- Active policy content (YAML string), version tag, last modified timestamp, and Git author metadata.
- Catalog of discovered MCP servers and registered tool names for autocomplete.

**Data Flow:**
1. Screen mounts → load active policy from `/api/v1/policies/active`.
2. Populate YAML editor and initialize JSON schema linter.
3. User edits rules → editor runs real-time local syntax validation.
4. User clicks "Test Rule" → send simulation payload to `/api/v1/policies/simulate`, displaying evaluation outcome in simulator panel.
5. User clicks "Deploy" → open Deploy Confirmation Modal; on confirm, send `PUT /api/v1/policies/active`.

### States

| State | Condition | What the User Sees |
|---|---|---|
| **Clean** | Editor matches deployed policy | "Policy Active & Synced" green indicator |
| **Draft Dirty** | Uncommitted edits in editor | Amber indicator: "Unsaved Changes" with "Diff" and "Deploy" enabled |
| **Lint Error** | Invalid YAML syntax or unknown tool keyword | Inline red wavy underline in editor; error banner below editor with line number |
| **Simulating** | Running test payload against rules | Spinner in simulation panel; displays green "ALLOWED" or red "BLOCKED" badge |
| **Deploying** | Submitting policy update | Deploy button shows spinner; editor inputs temporarily disabled |

**Edge Cases:**
- Deploying a policy that blocks all tools cluster-wide → Display high-severity warning modal: "This policy blocks 100% of currently connected agents. Are you sure?"
- GitOps managed mode enabled → Disable direct web deployment; replace "Deploy" button with "Generate Git Commit / PR" instructions.
