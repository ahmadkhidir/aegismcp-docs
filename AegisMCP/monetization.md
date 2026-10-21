# Revenue Model & Monetization

## Revenue Model

**Model Type:** Open-Core / Tiered Subscription (Community Free, Team Pro, Enterprise Dedicated)

**Summary:**
AegisMCP monetizes through an open-core commercial model. Individual software engineers and AI developers use the self-hosted open-source core proxy engine and CLI for free to secure local agent tools. Engineering teams and regulated enterprises pay a tiered subscription for centralized multi-cluster policy synchronization, the Web Admin Console, Slack Human-in-the-Loop workflows, SIEM integrations, and automated EU AI Act / SOC 2 compliance reports.

---

## Pricing Tiers

### Community Tier (Open-Core)
**Price:** $0 / month (Perpetually free, Apache 2.0 license)
**Target:** David Chen (Senior AI Applications Engineer / Individual Developer)
**Purpose:** Developer acquisition and ecosystem adoption — establish AegisMCP as the standard runtime security boundary for MCP agents.
**Includes:**
- Core high-performance JSON-RPC streaming reverse proxy engine ([FEAT-001](features/FEAT-001.md))
- Local declarative YAML policy engine ([FEAT-002](features/FEAT-002.md))
- Developer command-line wrapper and inspector (`aegis-cli`) ([FEAT-003](features/FEAT-003.md))
- Standard outbound secret and PII scrubbing ([FEAT-005](features/FEAT-005.md))
- Local file-based audit write-ahead log (WAL) ([FEAT-007](features/FEAT-007.md))
- Local CLI emergency kill-switch ([FEAT-010](features/FEAT-010.md))

**Limitations:**
- Single-node / workstation operation only
- No Web Management Console or live browser streaming inspector
- No automated Slack/Teams Human-in-the-Loop interactive approvals
- No cryptographic audit integrity verification or compliance report generation

### Team Pro Tier
**Price:** $199 / month (or $1,990 / year, billed annually)
**Target:** Marcus Vance (Staff Platform Engineer at growth startups and mid-market SaaS)
**Purpose:** Core recurring revenue — provide platform and security teams with team-wide operational visibility and automated oversight.
**Includes:**
- Everything in Community, plus:
- Web Admin Security Console with live SSE traffic inspector ([FEAT-008](features/FEAT-008.md))
- Interactive Human-in-the-Loop approval gateway via Slack and webhook endpoints ([FEAT-006](features/FEAT-006.md))
- Advanced behavioral tool-chaining anomaly detection with custom sequence definitions ([FEAT-004](features/FEAT-004.md))
- Standard OIDC Bearer token authentication for agent workloads ([FEAT-009](features/FEAT-009.md))
- Automated circuit breaker thresholds and rate-limiting protections ([FEAT-010](features/FEAT-010.md))
- 30-day centralized audit log retention with SIEM streaming export
- Standard business hours email and community Slack support

### Enterprise Tier
**Price:** Custom enterprise contract (starting at $1,500 / month based on agent fleet size)
**Target:** Priya Sharma (CISO / Compliance Officer) and Elena Rostova (Lead DevSecOps)
**Purpose:** High-contract-value enterprise accounts in banking, healthcare, and defense requiring formal AI governance and compliance attestations.
**Includes:**
- Everything in Team Pro, plus:
- Mutual TLS with SPIFFE/SPIRE cryptographic workload identities ([FEAT-009](features/FEAT-009.md))
- Cryptographically sealed, tamper-evident audit ledger with continuous hash-chaining ([FEAT-007](features/FEAT-007.md))
- Automated compliance report generator for EU AI Act (Article 12 & 14) and SOC 2 Type II ([FEAT-011](features/FEAT-011.md))
- Dynamic tool parameter validation with deep SQL injection and path-traversal sanitization ([FEAT-012](features/FEAT-012.md))
- Multi-cluster fleet policy distribution and GitOps synchronization
- 365-day immutable audit log retention
- Enterprise Single Sign-On (SAML 2.0 / Okta) and granular role-based access control (RBAC)
- Guaranteed 99.99% uptime SLA ([NFR-010](nfr.md)), dedicated account manager, and 24/7 priority security support

---

## Feature Gate Mapping

| Feature ID | Feature | Community (Free) | Team Pro | Enterprise |
|---|---|---|---|---|
| [FEAT-001](features/FEAT-001.md) | Zero-Trust JSON-RPC Streaming Reverse Proxy | Yes | Yes | Yes |
| [FEAT-002](features/FEAT-002.md) | Declarative Policy-as-Code Engine for MCP Tools | Local YAML | Local + GitOps | Central Fleet Sync |
| [FEAT-003](features/FEAT-003.md) | Local Developer CLI Proxy (`aegis-cli`) | Yes | Yes | Yes |
| [FEAT-004](features/FEAT-004.md) | Behavioral Tool-Chaining Anomaly Detector | Basic heuristics | Full sequence rules | Custom enterprise rules |
| [FEAT-005](features/FEAT-005.md) | Outbound Tool Response Secret & PII Scrubber | Standard rules | Custom regex rules | Enterprise DLP engine |
| [FEAT-006](features/FEAT-006.md) | Human-in-the-Loop (HITL) Out-of-Band Approval Gateway | No | Slack + Webhook | Multi-channel + Custom SLA |
| [FEAT-007](features/FEAT-007.md) | Cryptographic Tamper-Evident Audit Ledger | Local disk WAL | 30 days retention | 365 days + Crypto Sealing |
| [FEAT-008](features/FEAT-008.md) | Web Admin Security Console & Live Traffic Inspector | No | Up to 5 admin seats | Unlimited seats + SSO |
| [FEAT-009](features/FEAT-009.md) | Agent Identity Verification & SPIFFE/OIDC Integration | Basic token | OIDC Bearer JWT | SPIFFE/SPIRE mTLS + IdP |
| [FEAT-010](features/FEAT-010.md) | Emergency Global Kill-Switch & Circuit Breaker | CLI only | Web Console + CLI | Global Fleet Lockdown API |
| [FEAT-011](features/FEAT-011.md) | Automated Compliance Report Generator (EU AI Act & SOC 2) | No | No | Yes (Signed PDF & JSON) |
| [FEAT-012](features/FEAT-012.md) | Dynamic Argument Schema Validator & Sanitizer | No | Schema validation | Deep Injection Sanitizer |

---

## Payment & Billing

**Payment Processor:** Stripe for credit card and self-serve Pro subscriptions; Stripe Invoicing / ACH Wire for Enterprise annual agreements.
**Billing Cycle:** Monthly or Annual (annual billing includes a 17% discount, offering two months free).
**Trial Period:** 14-day fully featured free trial of Team Pro tier with no credit card required; Community tier is free forever.
**Currency Support:** Charged in USD; localized pricing displays available in EUR and GBP for European customers.
**Tax Handling:** Automated calculation via Stripe Tax for US state sales tax, EU VAT (with reverse-charge B2B validation), and UK VAT.
**Dunning:** Automated 4-step retry over 14 days before account suspension. Email notifications sent to billing contacts on days 1, 3, 7, and 14.
**Invoices & Receipts:** PDF invoices automatically delivered via email and downloadable from the account settings view.

**Billing Events:**
- **Upgrade:** Immediate charge prorated for the remaining billing cycle duration; new features unlock immediately.
- **Downgrade:** Changes take effect at the end of the current billing cycle; access remains active until cycle completion.
- **Cancellation:** Subscription cancels at end of current paid period; gateway deployments revert to Community tier capabilities.
- **Failed Payment:** Account enters a 14-day grace period with email alerts; if unresolved, enterprise features suspend to Community tier.
- **Refund:** Discretionary full refund within 14 days of initial purchase upon request; pro-rated refunds for enterprise service interruptions violating SLA.

---

## Billing Operations & Fraud Policy

| Area | Policy |
|---|---|
| Refund window | Full refund within 14 days of purchase upon written request; non-refundable thereafter |
| Chargeback response | Immediate notification; contested with access logs and signed terms of service; immediate account suspension upon dispute |
| Failed-payment grace | 14 calendar days of uninterrupted service before account features are restricted |
| Fraud triggers | Multiple card declines (> 3 within 1 hour), disposable email domains on paid tiers, card country mismatch with IP |
| Fraud response | Automated payment block, account flagged for security review, notification sent to account owner |
| Proration | Exact second-level proration applied to mid-cycle upgrades and tier additions |

---

## Key Metrics

| Metric | Definition | Target |
|---|---|---|
| Community-to-Paid Conversion | % of active open-core installations converting to paid tiers within 90 days | > 5.0% |
| Monthly Recurring Revenue (MRR) | Aggregate monthly subscription revenue | > $50,000 within 12 months |
| Net Revenue Retention (NRR) | Expansion revenue minus churn from existing customers over 12 months | > 120% |
| Annual Churn Rate | % of paying customers cancelling subscription annually | < 4.0% |
| Customer Acquisition Cost Payback | Time required to recover acquisition costs from customer margin | < 9 months |
| Average Revenue Per Account (ARPA) | Total monthly revenue divided by number of active paid accounts | > $650 / month |

---

## Monetization Roadmap

| Phase | Roadmap Phase | Monetization Milestone |
|---|---|---|
| 1 | [Phase 1: Zero-Trust Security Foundation](roadmap.md) | Release Community Open-Core edition; open waitlist and design partnership program for Team Pro |
| 2 | [Phase 2: Operational Visibility](roadmap.md) | Launch self-serve Team Pro tier ($199/mo) with Stripe billing and Web Console |
| 3 | [Phase 3: Compliance Scale](roadmap.md) | Launch Enterprise sales motion with custom contracts, compliance reporting, and annual billing |
