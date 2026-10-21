# AegisMCP

**One-liner:** An enterprise-grade Zero-Trust reverse proxy and security firewall that intercepts, policies, and audits Model Context Protocol (MCP) tool calls between AI agents and backend systems.
**Status:** In Documentation
**Interfaces:** API/Service, CLI, Web

## Overview

AegisMCP is an enterprise-grade Zero-Trust reverse proxy and security firewall for autonomous AI agents using the Model Context Protocol (MCP). It intercepts JSON-RPC tool calls, resource accesses, and prompt templates over HTTP/SSE, WebSocket, and standard I/O streams in real time. The platform evaluates declarative security policies, detects multi-step tool-chaining anomalies, scrubs outbound secrets and personal data, and enforces human-in-the-loop approvals before high-impact tools execute against backend infrastructure.

## The Problem

Enterprises deploying autonomous AI agents face critical security vulnerabilities when granting them tool execution rights via the Model Context Protocol. Without runtime mediation, an agent compromised by indirect prompt injection can execute destructive operations, access unapproved internal systems, or leak sensitive data through arbitrary tool parameters. Security teams lack visibility, centralized authorization, and immutable audit logging for non-deterministic agent tool invocations, forcing organizations to either block agent adoption or accept unmitigated breach risks.

**How it's solved today:** Developers and platform teams currently rely on unrestrained direct connections where agents inherit the full ambient privileges of host environments, write ad-hoc Python wrappers around individual tools, or attempt prompt-level system instructions that models routinely bypass under injection attacks.

## The Solution

AegisMCP places a high-throughput, non-blocking Zero-Trust proxy inline between AI agent hosts and MCP tool servers to inspect every protocol message before execution. It enforces fine-grained Policy-as-Code to restrict accessible tools per agent identity, validates parameter schemas against injection payloads, detects dangerous chained behaviors, and requires explicit out-of-band approvals for sensitive actions. Real-time data loss prevention sanitizes tool outputs before returning them to the model context, while a tamper-evident audit ledger provides complete provenance for compliance frameworks.

## Market & Competition

**Alternatives today:**
- [ressl/mcp-firewall](https://github.com/ressl/mcp-firewall) — open-source single-process CLI wrapper in Python for local stdio MCP servers; lacks distributed multi-tenancy, Kubernetes sidecar deployment, high-throughput streaming concurrency, and centralized policy sync.
- [luckypipewrench/pipelock](https://github.com/luckypipewrench/pipelock) — developer utility for agent data exfiltration protection; narrow scope without enterprise identity integration, stateful sequence tracking, or human approval workflows.
- [Praesidia](https://praesidia.ai) — early-stage closed-source platform addressing tool poisoning and prompt injection in agent ecosystems.
- [Solo.io agentgateway](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHYfmJMavKEvoqWw240whI8Bn9ZjSUrGt3I8Ymk-1rxGfDnk9Ys2UQNItI3HkOnTp3TcC15UgwXtBOfY1LjD_CC8Gp5r9QXH0MslhuTX0D7i81UpprJSyC6p9BNdN0dtE6cWU6sfORSReqrDb3kZ2VINYOkPleIvQ==) — enterprise API gateway extension for AI workloads focused on network routing rather than deep MCP semantic firewalling.
- Direct unmediated connection — status quo default where agents communicate directly with MCP servers without access controls or audit records.

**Demand signal:** [OWASP Top 10 for LLMs and Generative AI (2025/2026 update)](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGxX7y37w4wtu8OaSq1o8v67rYdAjoNJtKQavWH7cXP5Mg5R6YR5ZlPak_GbwE2yIIPAfOlDUW1ts6HtyG0P-xDkHigHXcd7kVZR_8_IZsyILqvkdnUd5FwezrnpBMMWTx8es6EhMy07NTbJ5i9HSwqKriRd1NRrzED6KRgMhcZsk4W2BGo3jrtDJ2IEe5B1_o=) elevated LLM03: Excessive Agency into the top three enterprise vulnerabilities, driven by production agent tool adoption without perimeter controls.
**Platform/regulatory exposure:** [EU AI Act Article 14 (Human Oversight)](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHgo9cSCoTbgkp7oZgk6zNXFknU2EzVhXEYO9kuEW6SY6-HZWxvA6weK0sBd5ILhtwL0luT3yVYwpN9YIpGgzjIcuq73TFyNRV9Y5QI4CPNIh-3yr5pvkl285Ww24dgZvp_FlzTsv4JuDakJgzFgR0c) mandates technical mechanisms enabling real-time human intervention and operation prevention for high-risk autonomous AI systems.
**Verdict:** Proceed — The transition from text chatbots to tool-calling autonomous agents creates an immediate, unaddressed requirement for enterprise-grade, high-throughput Zero-Trust MCP gateways.

## Target Audience

**Primary:** Platform Engineers and DevSecOps teams responsible for deploying and securing autonomous agent workloads in enterprise cloud environments.
**Secondary:** Security Operations (SecOps) and Compliance Officers needing auditable records of AI actions to meet SOC 2, ISO 42001, and EU AI Act mandates.
**Not for:** Consumer end-users interacting with isolated conversational chatbots that execute no external tools.

These users manage cloud infrastructure where unauthorized tool calls risk data breaches, service outages, or compliance penalties, and need a drop-in security gateway that integrates into existing identity and monitoring stacks.

## Success Metrics & KPIs

| Metric | Definition | Baseline | Target | Timeframe | Source |
|---|---|---|---|---|---|
| Proxy latency overhead (P99) | Added round-trip latency per inspected JSON-RPC tool invocation | — | < 5 ms | At launch | Gateway Telemetry |
| Policy evaluation accuracy | % of unauthorized or policy-violating tool calls successfully blocked | — | 100% | At launch | Test Harness |
| Enterprise activation rate | % of pilot organizations deploying AegisMCP to protect production agent tools | — | > 35% | 90 days post-launch | Usage Analytics |
| High-risk action interception | % of destructive tool calls routed to human approval | — | 100% | At launch | Audit Ledger |

## SWOT Analysis

### Strengths *(internal, positive)*
- Built on modern Java 21 with Virtual Threads (Project Loom), delivering sub-millisecond P99 latency and high-throughput non-blocking proxying for tens of thousands of concurrent agent streams.
- Protocol-native deep inspection designed specifically for JSON-RPC 2.0 over HTTP/SSE, WebSocket, and standard I/O.
- Built-in state machine for multi-step tool-chaining anomaly detection, catching complex confused-deputy attacks that single-request firewalls miss.
- Enterprise-ready integration surface including SPIFFE/SPIRE agent identities, OAuth2/OIDC, and OpenTelemetry instrumentation.

### Weaknesses *(internal, negative)*
- Lean initial engineering bandwidth limits simultaneous delivery of deep GUI consoles alongside the core high-performance proxy engine.
- Zero existing enterprise brand awareness compared to incumbent API gateway vendors.
- Relying on community contributions or user-authored rule packs for niche, domain-specific MCP tool schemas.

### Opportunities *(external, positive)*
- Rapid industry convergence on the Model Context Protocol (MCP) as the universal standard for AI tool integrations.
- Regulatory mandates (EU AI Act, NIST AI RMF) penalizing enterprises for uncontrolled autonomous AI operations and lack of human oversight.
- Incumbent API gateways treat LLM traffic as standard REST/HTTP endpoints, leaving semantic JSON-RPC tool inspection and stateful agent chaining unaddressed.

### Threats *(external, negative)*
- Upstream MCP specifications could introduce breaking protocol changes to JSON-RPC transports.
- Large security incumbents may add native MCP proxy capabilities to their existing edge networks.
- Model providers could advance native internal model alignment, reducing perceived need for external tool call firewalls among casual developers.

## Scope

**In scope:**
- High-performance reverse proxy for MCP JSON-RPC 2.0 over HTTP/SSE, WebSocket, and stdio streams.
- Policy-as-Code engine evaluating tool allow/block lists, parameter schema validation, and path/regex constraints.
- Behavioral sequence tracking to detect and block malicious tool-chaining patterns across agent sessions.
- Human-in-the-Loop (HITL) approval gateway for high-impact actions via webhooks (Slack, email, dashboard).
- Real-time outbound data sanitization (PII masking and API credential scrubbing) on tool response payloads.
- Tamper-evident cryptographic audit ledger for all tool requests, evaluations, and execution results.

**Out of scope (for now):**
- Hosting or serving LLM weights (e.g., vLLM or Ollama execution).
- Multi-region geo-distributed policy consensus clustering (deferred to Phase 2 enterprise cloud).
- Automatic natural-language prompt synthesis or automated agent prompt optimization.

**Explicit non-goals:**
- AegisMCP will never act as an agent orchestrator or execution framework (it does not plan or prompt agents; it restricts and audits them).
- AegisMCP will not replace network-level firewalls or general HTTP WAFs for generic web traffic.

## Assumptions & Dependencies

**Assumptions** — believed true; if wrong, the plan changes:
| # | Assumption | Risk if false | Validated? |
|---|---|---|---|
| A1 | MCP continues as the dominant open standard for AI agent tool integration across enterprise tools. | Slower enterprise adoption if competing tool protocols fragment the market. | Yes — Anthropic, Cursor, and enterprise ecosystem adoption. |
| A2 | Security and platform engineering teams will accept a dedicated proxy layer if latency overhead remains under 5ms. | Adoption friction if developers bypass the gateway due to latency concerns. | No — Requires benchmark validation on Java 21 engine. |
| A3 | Enterprises require on-premises and private VPC deployments for regulatory compliance rather than purely hosted SaaS. | Product architecture must prioritize containerized self-hosted binaries over SaaS-only tenancy. | Yes — Mandated by banking and healthcare security policies. |

**Dependencies** — external and blocking, not under the team's control:
| # | Dependency | Type | Blocks | Status |
|---|---|---|---|---|
| D1 | Model Context Protocol (MCP) Specification stability | Protocol standard | Core parser stability | Active — tracked at modelcontextprotocol.io |
| D2 | Java 21 LTS runtime availability in customer enterprise container base images | Runtime platform | Deployment | Active — standard in modern enterprise Linux distributions |

## Documentation

| Document | Purpose |
|---|---|
| [overview](overview.md) | Root product definition, market validation, scope, and strategy |
| [user-personas](user-personas.md) | Target audience personas and stakeholder requirements |
| [branding](branding.md) | Brand identity, design character, mark system, and design tokens |
| [features](features.md) | Prioritized feature catalog, user stories, and acceptance criteria |
| [nfr](nfr.md) | Non-functional quality targets: performance, security, availability, and compliance |
| [roadmap](roadmap.md) | Phased product delivery plan and milestone roadmap |
| [architecture](architecture.md) | System components, data flows, deployment topology, and security controls |
| [monetization](monetization.md) | Open-core revenue model, pricing tiers, feature gates, and billing operations |
| [risk-log](risk-log.md) | Assessed risks derived from SWOT, mitigation plans, and review cadence |
| [user-flow](user-flow.md) | End-to-end user journeys, walkthroughs, decision points, and navigation flows |
| [screens](screens.md) | Web management console screens, navigation map, and shared components |
| [schema](schema.md) | Data schemas, entity definitions, JSON Schema 2020-12 blocks, and relationships |
| [analytics](analytics.md) | Telemetry taxonomy, security conversion funnels, error monitoring, and dashboards |
| [operations](operations.md) | Support SLAs, administrative governance, billing operations, and incident runbooks |
| [release](release.md) | Promotion gates, rollout phases, feature flags, rollback triggers, and go-live checklist |
