# The KafClaw Ecosystem: Production-Grade Agentic Infrastructure

The KafClaw ecosystem provides the full vertical stack required to run AI agents in enterprise production. This is not a collection of demos or wrappers. It is a coordinated set of projects covering agent coordination, version control, event transport, and private AI governance.

Each project is independently useful. Together they form a complete operational layer for autonomous agent deployments.

---

## Core Pillars

1. **Agent Runtime (KafClaw)**: The full runtime layer for autonomous agents. Handles the agent loop, LLM abstraction, tool execution, memory, scheduling, and channel integrations, while the Kafka group layer connects fleets of agents into observable, hierarchical swarms.
2. **Infrastructure (GitClaw)**: Agent-first Git service. A hard fork of Gitea that treats machine identities as first-class participants in repository workflows, not exceptions to be worked around.
3. **Event Transport (KafScale)**: A Kafka-protocol compatible streaming platform with stateless brokers and S3-backed storage. Handles transport and durability only; stream processing is delegated to external compute engines like Apache Flink or Apache Wayang.
4. **Governance (Scalytics Copilot)**: Private AI operations stack. Governed, auditable, self-hosted inference with role-based access controls across local and external models.

---

## The Projects

### [KafClaw](https://github.com/KafClaw/KafClaw) - Full Agent Runtime

KafClaw is a Go-based agent runtime built on top of Apache Kafka. It is not a coordination layer sitting above existing agents. It is the runtime that agents actually run on, plus the group layer that connects multiple agents together into coordinated swarms.

**What it does locally:** Each KafClaw agent runs a full agent loop with LLM provider abstraction (OpenAI, OpenRouter), a tool registry with security sandboxing (filesystem, shell, memory, web), a policy engine for message classification and token quotas, a scheduler for cron and deferred tasks, a SQLite-backed timeline for event logging and distributed tracing, and channel bridges for WhatsApp, Telegram, Discord, and a web UI.

**What it does across agents:** Agents communicate via a structured Kafka topic protocol using typed envelopes (`announce`, `request`, `response`, `trace`, `memory`, `audit`) with correlation IDs and timestamps. Every interaction is traceable and replayable by design.

**Group collaboration:** Agents form groups. Within a group, they discover each other via heartbeats, delegate tasks through request/response topics, share memory, and register skills as addressable services. Cross-group coordination is handled by an orchestrator layer that adds hierarchy, zone-based security boundaries, and real-time discovery.

**Language independence:** Any process that can produce and consume Kafka envelopes participates in the group. A Go binary running the full KafClaw runtime, a Python script using `kafka-python`, a Node.js service, or a shell script polling via `kcat` all speak the same wire protocol. No SDK lock-in beyond Kafka itself.

**Operating modes:** `standalone` (no Kafka, single-agent desktop use), `group` (peer-to-peer agent collaboration), `full` (hierarchical multi-agent with orchestrator and zones), and `headless` (server deployment with bearer token auth).

**Shared memory:** Agents publish knowledge artifacts to a shared memory topic backed by S3 or LFS for large payloads. Each subscribing agent indexes received items locally for semantic retrieval, enabling agents with different architectures and languages to build on each other's work without direct coupling.

**Built on:** Inspired by [HKUDS/nanobot](https://github.com/HKUDS/nanobot) (MIT) and completely written in Go. Licensed Apache 2.0. 

---

### [GitClaw](https://github.com/KafClaw/GitClaw) - Agent-First Git Service

GitClaw is a hard fork of Gitea designed for autonomous machine-to-machine Git workflows. It extends Gitea's human-facing web UI with a first-class agent enrollment and policy model.

**What it adds:** A public agent enrollment endpoint (`POST /api/v1/agents/enroll`) guarded by server policy, with CIDR allow lists for source network controls. Enrolled agents receive stable, normalized bot account identities and tokens. Token rotation is handled automatically on re-enrollment, eliminating manual credential management.

**Key features:** Optional auto-creation of a bootstrap repository per enrolled agent. Skills-based onboarding via public `skill.md` and `scripts/enroll.sh`. No use of `INTERNAL_TOKEN` for external requests. Enrollment trust is policy-based, controlled through endpoint configuration and admin controls.

**Security posture:** External requests never touch `security.INTERNAL_TOKEN`, which remains a server-internal primitive. Enrollment trust is enforced through endpoint policy and CIDR filtering rather than shared secrets.

Licensed MIT. Based on Gitea, evolving independently for agentic use cases. 

---

### [KafScale](https://kafscale.io) - Kafka-Protocol Streaming on S3

KafScale is a Kafka-protocol compatible streaming platform built on a single architectural premise: durable logs belong in object storage, not in stateful brokers. It is implemented in Go and licensed Apache 2.0.

**What it does:** Brokers are stateless and ephemeral. They handle Kafka protocol traffic and buffer segments in memory, but hold no durable state of their own. S3 stores immutable log segments and index files as the source of truth. etcd manages metadata, consumer group state, and offsets. The Kafka protocol surface remains intact, so existing producers and consumers work without modification.

**What it does not do:** KafScale is a transport and storage layer, not a compute engine. Stream processing is expected to run in external engines such as Apache Flink or Apache Wayang. KafScale moves and retains the data; those engines process it.

**Why this matters for agents:** Traditional Kafka broker clusters require persistent disk, leader election, rebalancing, and overprovisioning to survive failures. KafScale eliminates those structural costs. Broker instances are disposable. Storage scales independently. Replay is available as long as the S3 objects exist, with no broker state to protect.

Repository: [KafScale/platform](https://github.com/KafScale/platform). Docs: [kafscale.io](https://kafscale.io).

---

### [Scalytics Copilot](https://github.com/scalytics/ScalyticsCopilot) - Private AI Operations

Scalytics Copilot is an open source private AI operations stack for organizations that cannot run on unmanaged, black-box AI tooling. It handles local LLM deployment, model routing, user and group permissions, and API key governance in a single self-hosted platform.

**What it does:** Runs LLMs in the 7B to 70B range locally via vLLM with an OpenAI-compatible API surface. Routes requests across local and external providers based on policy, not hardcoded configuration. Provides a web dashboard, chat UI, and admin panel with usage monitoring and hardware visibility.

**Who it is for:** Enterprise teams that need governed AI operations across departments, compliance requirements around data locality, or deployment ownership that third-party SaaS cannot provide.

**Stack:** Node/Express backend, React frontend, Python services for inference and deep research. Supports MCP tools and agent workflow integration. Deployment scripts for Linux and GCP included.

Licensed Apache 2.0. Backed by [Scalytics](https://www.scalytics.io).

---

## More from the same team

- [KafGraph](https://github.com/scalytics/KafGraph): a shared knowledge graph for agents. It runs as a KafScale processor and reads the KafClaw group topics.
- [kafSIEM](https://github.com/scalytics/kafSIEM): an entity graph over Kafka agent traffic and OSINT feeds. KafClaw agents are one of its inputs.
- [kshark-core](https://github.com/scalytics/kshark-core): Kafka connectivity diagnostics, and an example of how agents can use our tools to debug Kafka clusters.
- [infinimesh](https://infinimesh.org/): a self-hosted IoT device platform. Its Kafka addon streams device data to Kafka or KafScale.

All projects are listed at [scalytics.io/open-source](https://www.scalytics.io/open-source). For governed AI agents on Kafka with commercial support, see [Lascaris](https://www.scalytics.io/lascaris).

Built by the Scalytics team, including [Alexander Alten](https://www.novatechflow.com/p/about-us.html). Background: [how I built a secure, high-performance AI agent squad](https://www.novatechflow.com/2026/02/how-i-built-secure-high-performance-ai.html).

---

## Why This Exists

Most agent frameworks are built around a single human controller in a stable, low-latency environment. Enterprise environments are distributed, heterogeneous, and subject to audit requirements that most frameworks do not address.

The KafClaw ecosystem was built to provide the four things production agent deployments actually need: interoperability across languages and runtimes, full observability into why an agent took an action, machine-identity-based access control that does not require human intervention, and the throughput headroom to handle what happens when autonomous agents start generating work at scale.

KafClaw is backed by [Scalytics](https://www.scalytics.io). The ecosystem does not create, operate, or endorse any crypto tokens. If you see token-based fundraising using the KafClaw name, it is not affiliated with this project.
