# Features & Capabilities

## Overview

AegisMCP delivers a comprehensive, enterprise-grade security and governance layer for autonomous AI agents interacting with backend systems via the Model Context Protocol (MCP). The feature set establishes deterministic Zero-Trust perimeter security, real-time prompt injection and data exfiltration defense, mandatory Human-in-the-Loop oversight for high-risk operations, and tamper-evident compliance auditability.

---

## Priority Definitions

| Priority | Label | Meaning |
|---|---|---|
| **P0** | Must Have | Critical for launch. Core security, proxying, policy enforcement, and audit ledger. |
| **P1** | Should Have | Important operational capabilities. Web console, agent identity integration, and CLI debugging. |
| **P2** | Could Have | Enterprise enhancements. Compliance export reports and advanced payload sanitization. |
| **P3** | Won't Have Now | Acknowledged but deferred to post-1.0 enterprise roadmap. |

---

## Feature Inventory

### Core Protocol Proxy & Traffic Interception

| ID | Feature | File | Priority |
|---|---|---|---|
| FEAT-001 | Zero-Trust JSON-RPC Streaming Reverse Proxy | [FEAT-001](features/FEAT-001.md) | P0 |
| FEAT-002 | Declarative Policy-as-Code Engine for MCP Tools | [FEAT-002](features/FEAT-002.md) | P0 |
| FEAT-003 | Local Developer CLI Proxy (`aegis-cli`) | [FEAT-003](features/FEAT-003.md) | P1 |

### Real-Time Threat Detection & Data Protection

| ID | Feature | File | Priority |
|---|---|---|---|
| FEAT-004 | Behavioral Tool-Chaining Anomaly Detector | [FEAT-004](features/FEAT-004.md) | P0 |
| FEAT-005 | Outbound Tool Response Secret & PII Scrubber | [FEAT-005](features/FEAT-005.md) | P0 |

### Governance, Human-in-the-Loop & Auditability

| ID | Feature | File | Priority |
|---|---|---|---|
| FEAT-006 | Human-in-the-Loop (HITL) Out-of-Band Approval Gateway | [FEAT-006](features/FEAT-006.md) | P0 |
| FEAT-007 | Cryptographic Tamper-Evident Audit Ledger | [FEAT-007](features/FEAT-007.md) | P0 |
| FEAT-008 | Web Admin Security Console & Live Traffic Inspector | [FEAT-008](features/FEAT-008.md) | P1 |
| FEAT-009 | Agent Identity Verification & SPIFFE/OIDC Integration | [FEAT-009](features/FEAT-009.md) | P1 |
| FEAT-010 | Emergency Global Kill-Switch & Circuit Breaker | [FEAT-010](features/FEAT-010.md) | P1 |

### Compliance & Enterprise Extensibility

| ID | Feature | File | Priority |
|---|---|---|---|
| FEAT-011 | Automated Compliance Report Generator (EU AI Act & SOC 2) | [FEAT-011](features/FEAT-011.md) | P2 |
| FEAT-012 | Dynamic Argument Schema Validator & Sanitizer | [FEAT-012](features/FEAT-012.md) | P2 |

---

## Priority Summary

| Priority | Count | Feature IDs |
|---|---|---|
| P0 | 6 | FEAT-001, FEAT-002, FEAT-004, FEAT-005, FEAT-006, FEAT-007 |
| P1 | 4 | FEAT-003, FEAT-008, FEAT-009, FEAT-010 |
| P2 | 2 | FEAT-011, FEAT-012 |
| P3 | 0 | None |
