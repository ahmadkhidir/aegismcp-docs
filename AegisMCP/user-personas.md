# User Personas

## Overview

AegisMCP serves technical and security professionals responsible for deploying, protecting, and governing autonomous AI agents in production environments. The target audience is segmented along operational responsibilities: infrastructure platform engineers ensuring low-latency deployment, DevSecOps engineers enforcing real-time threat prevention, agent application developers needing fast compliance, and enterprise compliance stakeholders demanding verifiable audit trails.

---

## Persona 1: Marcus Vance

**Tagline:** Platform engineer deploying secure, low-latency AI agent infrastructure across enterprise Kubernetes clusters.
**Demographic:** Age 34, San Francisco CA, Staff Platform Engineer at a high-growth B2B fintech (450 engineers).
**Role:** Staff Platform Engineer / AI Infrastructure Lead
**Tech Comfort:** High
**Research basis:** Cloud Native Computing Foundation (CNCF) platform engineering surveys and production discussions on containerized AI agent sidecar architectures.
**In their words:** "Our product teams want to give autonomous agents direct access to staging databases and internal microservices via MCP, but I cannot let a rogue prompt injection drop our tables or leak internal secrets."

### Goals
- Deploy a transparent, non-blocking reverse proxy sidecar that intercepts agent MCP connections with zero application code changes.
- Manage declarative, GitOps-driven Policy-as-Code (YAML) to strictly restrict available tools and argument schemas per service account.
- Enforce strict P99 latency overhead under 5ms so engineering teams encounter zero perceptible slowdown.
- Monitor connection health, streaming JSON-RPC throughput, and protocol error rates across thousands of concurrent agent sessions.

### Frustrations
- MCP servers default to ambient host permissions, exposing local environments, environment variables, and filesystem paths.
- Developers run unmanaged, ad-hoc MCP servers with personal credentials that bypass corporate network inspection.
- Existing API gateways (Kong, Envoy) treat traffic as generic HTTP and lack native support for JSON-RPC 2.0 streaming and agent sequence state.

### Behaviors
- Manages infrastructure using Kubernetes, Helm, ArgoCD, Terraform, Docker, and Linux system tools.
- Works primarily in the terminal and CLI on macOS and Linux workstations during weekday business hours.
- Reviews pull requests for infrastructure configuration and evaluates tooling through automated performance benchmarks.

### Scenario
When multiple engineering teams request clearance to connect autonomous IDE agents and backend worker bots to internal staging databases via MCP, Marcus deploys AegisMCP as a Kubernetes sidecar. He configures a GitOps policy restricting filesystem tool paths to `/workspace` and database tools to read-only `SELECT` statements with row limits. When an agent attempts an unapproved database migration command during an automated test, AegisMCP terminates the tool call at wire speed and logs the violation without impacting the developer's session.

---

## Persona 2: Elena Rostova

**Tagline:** DevSecOps lead mitigating agentic prompt injection, confused-deputy attacks, and data exfiltration.
**Demographic:** Age 38, London UK, Lead DevSecOps Engineer at an enterprise HealthTech SaaS (1,200 employees).
**Role:** Lead DevSecOps / Product Security Engineer
**Tech Comfort:** High
**Research basis:** OWASP GenAI Security Project (LLM03 Excessive Agency research) and DevSecOps working group findings on real-time data loss prevention in agent pipelines.
**In their words:** "Prompt injection is inevitable. The real battle is preventing the agent from acting as a confused deputy and exfiltrating patient PII or cloud credentials when it executes tools."

### Goals
- Block indirect prompt injection attacks from manipulating autonomous agents into unauthorized mutations or credential exfiltration.
- Automatically redact sensitive personal data (PII) and cloud credentials from outbound tool responses before they re-enter model context.
- Require Human-in-the-Loop (HITL) approval via Slack or email webhooks for high-risk actions (financial transactions, data deletion, privileged API calls).
- Detect and quarantine behavioral tool-chaining anomalies (such as reading a configuration file followed immediately by an outbound HTTP request).

### Frustrations
- AI agents chain seemingly harmless read operations into dangerous exfiltration sequences that bypass single-request firewalls.
- Post-execution security logs only reveal breaches after data has already leaked to external endpoints.
- Generic prompt-based system instructions consistently fail under encoded or multilingual jailbreak techniques.

### Behaviors
- Analyzes alerts using Datadog, Splunk, GitHub Security, and centralized SIEM dashboards.
- Coordinates security incident response and tunes detection rules across corporate development pipelines.
- Configures webhook notifications and escalation paths for high-priority security events.

### Scenario
An engineer uses an autonomous agent to process an external vendor PDF containing a hidden indirect prompt injection. The malicious payload instructs the agent to read `/etc/secrets/token` using a filesystem MCP tool and transmit the contents via an HTTP POST tool. As the agent attempts to execute the second step of the sequence, AegisMCP's behavioral chaining detector identifies the data exfiltration pattern, halts execution, and dispatches an emergency approval alert to Elena's Slack channel with the full session trace.

---

## Persona 3: David Chen

**Tagline:** AI application developer building compliant, production-grade autonomous agent workflows for enterprise clients.
**Demographic:** Age 29, Austin TX, Senior AI Applications Engineer at an enterprise workflow automation startup (35 employees).
**Role:** Senior AI Applications Engineer / Agent Developer
**Tech Comfort:** High
**Research basis:** Model Context Protocol open-source developer communities, Claude Desktop integration forums, and LangChain/LangGraph agent development discussions.
**In their words:** "I want to build autonomous CRM agents that book meetings and update customer records, but enterprise prospects refuse to sign unless we prove our agents have zero-trust tool guardrails."

### Goals
- Integrate enterprise-grade authorization and runtime guardrails into AI agent pipelines without writing bespoke security wrappers for every tool.
- Run a lightweight local development proxy (`aegis-cli`) to inspect and debug JSON-RPC tool calls locally before pushing to production.
- Provide prospective enterprise buyers with verified compliance reports demonstrating deterministic tool execution boundaries.
- Rapidly configure fine-grained tool permissions for custom MCP servers using intuitive YAML syntax.

### Frustrations
- Enterprise procurement and security audits delay deals by three to six months due to unmitigated agent autonomy risks.
- Lack of local tooling to inspect JSON-RPC 2.0 payloads, request-response latencies, and tool argument schemas.
- Writing custom parameter validation, rate limiting, and sanitization logic for every new MCP server wastes weeks of engineering time.

### Behaviors
- Develops in Python, Java, and TypeScript; works daily with Claude Desktop, Cursor, Docker, and Postman.
- Active in developer Discord communities and GitHub open-source discussions.
- Iterates quickly in local development environments before deploying services to cloud staging clusters.

### Scenario
David is pitching an automated customer support agent to an enterprise banking client. During technical due diligence, the client's security team asks how David guarantees an agent cannot accidentally refund thousands of dollars or delete customer accounts. David connects his agents through AegisMCP, demonstrates how declarative policies restrict financial tools to transactions under $50 with mandatory human approval above that threshold, and presents the tamper-evident audit log. The client passes the vendor assessment in two weeks.

---

## Persona Comparison

| Need / Behavior | Marcus Vance | Elena Rostova | David Chen |
|---|---|---|---|
| Primary goal | Low-latency K8s sidecar proxy | Threat prevention & PII/secret scrubbing | Fast compliance & local tool debugging |
| Primary interface | CLI & Kubernetes Helm/GitOps | Web Security Console & Slack Webhooks | CLI & Web Developer Inspector |
| Usage frequency | Daily (deployment & maintenance) | Daily (monitoring & policy tuning) | Daily (development & testing) |
| Latency sensitivity | Extreme (< 5ms P99) | High (< 10ms P99) | Moderate (< 50ms) |
| Key frustration | Host privilege leakage & latency overhead | Indirect prompt injection & tool chaining | Long enterprise security sales cycles |

---

## Stakeholder Personas

### Stakeholder 1: Priya Sharma — Chief Information Security Officer (CISO)
**Role:** Enterprise Chief Information Security Officer & Risk Officer
**Goal:** Establish verifiable governance and compliance over enterprise AI agent deployments to satisfy regulatory requirements without blocking technical innovation.
**Needs from the system:** Comprehensive tamper-evident audit logs, cryptographic receipt verification, exportable compliance reports, and enterprise Single Sign-On (SSO) with RBAC.
**Research basis:** EU AI Act compliance mandates (Article 12 Record-Keeping and Article 14 Human Oversight) and SOC 2 Type II trust service criteria.
**Feeds:** Feeds auditability and data governance requirements into [nfr](nfr.md) and enterprise reporting controls into [operations](operations.md).

---

## Anti-Personas

### Anti-Persona 1: Casual Chatbot End-User
**Who:** Consumers using conversational AI mobile apps or web chatbots (such as standard ChatGPT or Claude consumer chat) for drafting text, casual queries, or personal assistance.
**Why AegisMCP is not for them:** These users interact with isolated language models that do not connect to external tools, databases, or local execution environments via the Model Context Protocol. AegisMCP is an infrastructure-level security proxy designed specifically for autonomous agent tool execution and provides no utility for unaugmented conversational chat.
