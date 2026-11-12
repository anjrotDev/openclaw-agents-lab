# OpenClaw Agents Lab

## Project Overview

**Project:** OpenClaw Agents Lab  
**Repository:** `openclaw-agents-lab`  
**Document:** `docs/project-overview.md`  
**Version:** 0.1  
**Status:** Active — Learning & Experimentation  
**Started:** November 2026  
**Maintainer:** Anjrot

---

## 1. Project Vision

OpenClaw Agents Lab is a personal research and development initiative focused on understanding, experimenting with, and building autonomous AI agent systems.

The project aims to explore how AI agents can use language models, tools, memory, external services, and specialized agents to perform useful tasks with varying degrees of autonomy.

Unlike a traditional software product, the initial objective is not to deliver a commercial application.

The primary objective is to **develop practical knowledge through experimentation**, establish reproducible agent architectures, and identify opportunities for useful automation.

The laboratory may eventually evolve into a commercially viable solution, but product development will follow demonstrated technical capabilities and measurable value.

**Guiding principle: Learn first, validate second, productize third.**

---

## 2. Project Background

The project originated from an exploration of locally hosted language models using Ollama and Open WebUI.

That experiment demonstrated how to:

- Download and execute language models locally.
- Understand model parameters, quantization, and context windows.
- Compare models based on response quality and performance.
- Expose models through local APIs.
- Access a locally hosted chat interface over a private network.
- Configure reverse proxies and local DNS.

The experiment also revealed limitations when executing language models on CPU-only hardware.

This motivated a transition toward cloud-hosted AI agents that use external model APIs while maintaining control over the agent execution environment.

OpenClaw was selected as the initial platform because it provides tools, memory, automation, multi-agent capabilities, and multiple interfaces for interacting with agents.

---

## 3. Primary Objectives

The laboratory has the following objectives:

1. Understand the OpenClaw architecture and execution model.
2. Deploy and administer OpenClaw on cloud infrastructure.
3. Explore agent identity, instructions, workspace organization, and persistent memory.
4. Understand how tools and skills are discovered, configured, authorized, and executed.
5. Experiment with different AI model providers and configurations.
6. Compare models based on cost, quality, reliability, and tool-calling capabilities.
7. Explore agent orchestration, delegation, and specialization.
8. Understand scheduling, background execution, and autonomous workflows.
9. Evaluate security boundaries and permission models.
10. Document experiments, discoveries, limitations, and architectural decisions.
11. Identify practical applications that can eventually become reusable products or services.

The laboratory prioritizes understanding the technology rather than accumulating features.

---

## 4. Working Principles

### 4.1 Learning Through Repetition

The environment is intentionally designed to be reproducible and disposable.

Installations may be repeated, configurations reset, and cloud instances terminated when doing so improves understanding.

The objective is to learn how the system works, not simply preserve a functioning installation.

Documentation must retain the knowledge necessary to reproduce validated experiments.

### 4.2 Incremental Experimentation

Each experiment should have:

- A clearly defined learning objective.
- An explanation of the relevant concepts.
- A minimal implementation.
- Observable results.
- A documented conclusion.

Avoid introducing architectural complexity before it becomes necessary for the experiment.

### 4.3 Controlled Autonomy

Agents should have sufficient autonomy to complete authorized tasks without requiring approval for every internal operation.

Routine activities may include:

- Reading and organizing workspace files.
- Maintaining internal documentation.
- Updating memory and preferences.
- Researching public information.
- Creating artifacts within an authorized workspace.

However, sensitive operations must require explicit authorization, including:

- Modifying system configuration or permissions.
- Installing software.
- Accessing sensitive credentials.
- Publishing external content.
- Sending communications to third parties.
- Performing financial transactions.
- Accessing production systems.

Agents should report relevant file modifications, including absolute file paths.

Behavioral instructions are not security boundaries. Technical access controls must be evaluated separately.

### 4.4 Evidence-Based Evaluation

Agent responses and self-reported actions are not automatically considered verified facts.

Important findings should distinguish:

- Observed behavior.
- Tool-generated evidence.
- Agent-reported actions.
- Documentation-based assumptions.
- Unverified hypotheses.

A successful agent response does not necessarily prove that every declared capability is operational.

### 4.5 Cost Awareness

The laboratory uses paid cloud infrastructure and AI APIs.

Experiments should account for:

- Cloud compute and storage.
- Input and output tokens.
- Cached tokens.
- Embedding generation.
- Background tasks.
- External tools and services.

Cost optimization should not compromise learning objectives or response quality.

More capable models may be used when appropriate, while economical alternatives should be evaluated for routine operations.

---

## 5. Initial Architecture

The initial environment is hosted on AWS EC2.

```text
Maintainer's Mac
       |
       | SSH
       |
       v
AWS EC2 — Ubuntu Linux
       |
       +-- OpenClaw Gateway
       |       |
       |       +-- Agent Runtime
       |       |
       |       +-- Tools and Skills
       |       |
       |       +-- Memory
       |       |
       |       +-- Automations
       |       |
       |       +-- Agent Sessions
       |
       +-- Persistent Workspace
       |
       +-- Local CLI / TUI
       |
       +-- Dashboard (loopback)
                 ^
                 |
             SSH Tunnel

External services:
       |
       +-- OpenAI API
       +-- Google Gemini API (planned)
       +-- Other integrations (future)
```

### 5.1 Infrastructure

| Component               | Initial Selection                   |
| ----------------------- | ----------------------------------- |
| Cloud provider          | AWS EC2                             |
| Instance type           | `t3.medium`                         |
| CPU                     | 2 vCPU                              |
| Memory                  | 4 GiB                               |
| Operating system        | Ubuntu Server 24.04 LTS             |
| Storage                 | 30 GiB encrypted EBS gp3            |
| Administration          | SSH                                 |
| Web access              | SSH tunnel                          |
| Gateway network binding | Loopback                            |
| Agent execution         | Direct host execution               |
| Sandbox                 | Disabled during initial exploration |
| Containerization        | Not installed initially             |

Infrastructure may change as experiments require additional capabilities.

### 5.2 AI Providers

The laboratory initially uses OpenAI models through the API.

Google Gemini is planned as a secondary provider.

Model selection is an operational decision and may vary by agent or task.

The initial deployment used GPT-6 Astra for configuration and subsequently began testing GPT-5.6 Sol for routine technical tasks.

### 5.3 Agent Architecture

The first controlled installation uses a single primary agent:

**Nexus 🥷**

Nexus is the personal assistant and initial coordinator for laboratory experiments.

Its responsibilities include:

- Research and analysis.
- Planning and organization.
- Technical experimentation.
- Documentation.
- Reporting findings.
- Future coordination of specialized agents.

Specialized agents will be introduced only after understanding how OpenClaw manages agent identity, sessions, delegation, tools, and permissions.

---

## 6. Security Model

The laboratory is isolated from production projects and infrastructure.

The current deployment intentionally retains broad local execution capabilities to support exploration.

This is an accepted laboratory risk, not a recommended production configuration.

### 6.1 Current Security Characteristics

- Dedicated EC2 instance.
- Dedicated AWS Security Group.
- SSH restricted to the maintainer's public IP.
- OpenClaw Gateway bound to loopback.
- Dashboard accessed through an SSH tunnel.
- No public OpenClaw Gateway port.
- No EC2 IAM role initially assigned.
- Agent running under the Ubuntu system user.
- Direct execution without an active sandbox.
- Broad execution tools available.
- The Ubuntu user has passwordless sudo privileges.

The last three characteristics create a significant security risk.

The laboratory must not contain production credentials or sensitive business data.

### 6.2 Future Security Experiments

The project will explore:

- Docker-based tool sandboxing.
- Workspace access restrictions.
- Tool policies.
- Execution approvals.
- Dedicated Linux users without sudo privileges.
- Filesystem isolation.
- Service hardening.
- Network restrictions.
- Agent-to-agent permissions.
- Session visibility.
- Secure credential management.

The laboratory will compare security mechanisms before selecting a recommended architecture for future deployments.

### 6.3 Disposable Infrastructure

Cloud instances may be terminated and recreated.

However, terminating an instance does not reverse external actions, leaked credentials, API usage, financial charges, or communications already sent.

These risks must remain controlled even in disposable environments.

---

## 7. Memory and Context Research

Memory is a major area of experimentation.

The project will investigate:

- Conversation context windows.
- Persistent workspace memory.
- Markdown-based memory files.
- Semantic search.
- Embeddings.
- Memory indexing.
- Memory consolidation.
- Context compaction.
- Retrieval quality.

### 7.1 Hybrid AI Architecture

The laboratory will evaluate the use of local models for auxiliary tasks while keeping powerful cloud models for reasoning and orchestration.

Potential responsibilities include:

| Responsibility                     | Candidate Execution             |
| ---------------------------------- | ------------------------------- |
| Main agent reasoning               | OpenAI or Gemini API            |
| Complex research and orchestration | Cloud LLM                       |
| Semantic memory embeddings         | Local model                     |
| Memory search                      | Local vector index              |
| Document classification            | Local or economical cloud model |
| Specialized routine tasks          | Task-specific model             |

Ollama, LM Studio, and llama.cpp are candidate local inference technologies.

The initial hybrid experiment will focus on local memory embeddings.

The objective is to understand the architecture, privacy implications, resource consumption, and retrieval quality.

Cost savings are a secondary consideration because embedding APIs are already relatively inexpensive.

---

## 8. Agent Tools and Skills

The project will investigate how OpenClaw extends agent capabilities through tools and skills.

Topics include:

- Built-in tools.
- Tool schemas.
- Tool execution.
- Tool permissions.
- Skills discovery.
- Custom skills.
- External integrations.
- Tool execution logs.
- Failure handling.
- Human approval.
- Untrusted tool output.

The distinction between an agent's declared capabilities and its actual permissions is a central learning objective.

Third-party skills must be reviewed before installation.

---

## 9. Multi-Agent Research

The initial environment uses a single agent.

Future experiments may introduce specialized agents with distinct roles, models, permissions, and workspaces.

Potential specialists include:

| Agent             | Responsibility                                       |
| ----------------- | ---------------------------------------------------- |
| Research Agent    | Investigate topics and produce source-backed reports |
| Content Agent     | Transform research into editorial drafts             |
| Planning Agent    | Organize objectives, tasks, and dependencies         |
| Analyst Agent     | Evaluate information and produce structured findings |
| Coordinator Agent | Delegate work and consolidate results                |

Research topics include:

- Agent creation and configuration.
- Agent-specific instructions.
- Tool access per agent.
- Shared versus isolated memory.
- Delegation mechanisms.
- Agent-to-agent communication.
- Session management.
- Execution monitoring.
- Cost allocation by agent.

The laboratory will not assume that every task requires multiple agents.

---

## 10. Automation and Background Execution

OpenClaw supports scheduled and autonomous operations.

The project will explore:

- Heartbeat.
- Cron jobs.
- Background execution.
- Scheduled research.
- Periodic reports.
- Notifications.
- Task monitoring.
- Retry and error handling.

During initial setup, the automatically configured heartbeat and memory dreaming tasks were disabled to avoid unnecessary API consumption.

These mechanisms will be reintroduced during dedicated experiments.

Autonomous operations must have a defined purpose, execution scope, and cost considerations.

---

## 11. Potential Business Application

### Affiliate Research & Content Automation

The first potential business application combines AI-powered market research, affiliate opportunity discovery, and SEO-oriented content production.

The objective is to create a system that helps website owners identify opportunities and prepare useful commercial content.

### 11.1 Initial Validation Website

**anjrot.com**

The existing website will serve as the initial validation environment.

### 11.2 Potential Affiliate Sources

- Amazon Associates for relevant physical products.
- Impact.com for software, hosting, VPN services, and other eligible offers.
- Mercado Libre for a possible future content website.

Affiliate links will initially be selected and verified manually.

### 11.3 Proposed Workflow

```text
Market and Keyword Research
          |
          v
Search Intent Analysis
          |
          v
Competitor Research
          |
          v
Product / Service Selection
          |
          v
Human Approval
          |
          v
SEO Content Research
          |
          v
Editorial Draft
          |
          v
WordPress Draft
          |
          v
Human Review and Publication
          |
          v
Performance Measurement
```

### 11.4 Content Principles

Content must:

- Provide genuine value to readers.
- Address identifiable search intent.
- Use verifiable sources.
- Avoid fabricated reviews or product testing.
- Clearly disclose affiliate relationships.
- Follow applicable affiliate program rules.
- Avoid mass-generated low-value content.
- Maintain editorial quality.
- Include human approval before publication.

### 11.5 Initial Success Indicators

Early validation will focus on:

- Search indexing.
- Organic impressions.
- Search query growth.
- Organic traffic.
- Affiliate link clicks.
- Content production costs.
- Editorial quality.

Longer-term validation will include approved commissions and profitability.

The project does not assume that organic traffic or affiliate revenue will grow immediately.

---

## 12. Future Productization Strategy

The project does not initially intend to build a traditional multi-tenant SaaS.

Instead, a successful workflow may be packaged as an installable AI agent solution.

The possible commercial offering would include:

- Preconfigured specialized agents.
- Custom tools and skills.
- Affiliate research workflows.
- SEO content workflows.
- WordPress integrations.
- Deployment documentation.
- Security recommendations.
- Monitoring and maintenance procedures.

Each customer would operate in an appropriately isolated environment with separate credentials, data, and permissions.

### Validation Sequence

1. Learn the platform.
2. Build a useful workflow.
3. Validate it on an existing website.
4. Measure organic traffic and commercial signals.
5. Improve the system using real data.
6. Repeat the experiment on a new website.
7. Evaluate productization only after demonstrating repeatable value.

**Commercialization is a potential outcome, not a requirement for the initial laboratory.**

---

## 13. Project Boundaries

### In Scope

- OpenClaw infrastructure and deployment.
- Agent configuration and architecture.
- Tools and skills.
- Memory and embeddings.
- AI model integration.
- Security and permissions.
- Automation and scheduling.
- Multi-agent experimentation.
- Technical documentation.
- Future affiliate content automation research.

### Out of Scope

- Abidegen infrastructure and repositories.
- Production business credentials.
- Customer onboarding during the learning phase.
- Multi-tenant SaaS development.
- Automatic commercial publishing during initial experiments.
- Unrestricted agent autonomy.
- Revenue guarantees.

---

## 14. Documentation Strategy

The laboratory will maintain independent documentation covering:

- Project overview.
- Roadmap.
- System architecture.
- Installation procedures.
- Security findings.
- Experiments.
- Technical decisions.
- Lessons learned.

Proposed repository structure:

```text
openclaw-agents-lab/
├── README.md
├── docs/
│   ├── project-overview.md
│   ├── roadmap.md
│   ├── architecture.md
│   └── installation/
│       └── aws-ec2-openclaw.md
└── experiments/
    ├── 001-agent-fundamentals/
    ├── 002-tools-and-skills/
    ├── 003-docker-sandbox/
    ├── 004-reproducible-openclaw-installation/
    └── 005-multi-agent/
```

Documentation should not contain API keys, authentication tokens, private SSH keys, or sensitive configuration values.

Only reviewed and sanitized configurations should be committed to Git.

---

## 15. Current Project State

As of November 8, 2026:

- AWS EC2 infrastructure is running.
- Ubuntu 24.04 LTS has been configured.
- OpenClaw has been installed successfully.
- Nexus has been initialized.
- OpenAI API integration is functional.
- The Dashboard is accessible through a private SSH tunnel.
- Initial identity and behavior files have been configured.
- Initial security audits have been performed.
- Broad host execution permissions have been identified.
- Heartbeat and memory dreaming were disabled during the initial experiments.
- GPT-5.6 Sol has been tested as an alternative to GPT-6 Astra.
- The project is evaluating memory, tools, skills, security, and local embeddings.

The laboratory remains in the experimentation phase.

---

## 16. Project Success Criteria

The learning phase will be considered successful when the maintainer can:

1. Deploy OpenClaw from a clean server.
2. Understand the role of the Gateway and agent runtime.
3. Configure agents and their persistent instructions.
4. Explain and control agent tools and permissions.
5. Configure more than one AI model provider.
6. Understand memory, context, and embeddings.
7. Create and test specialized agents.
8. Configure safe automated workflows.
9. Reproduce validated configurations from documentation.
10. Identify at least one useful real-world automation.

The future business phase will define separate technical and commercial success criteria.

---

## 17. Final Principle

**The laboratory exists to transform curiosity into practical knowledge, practical knowledge into useful automation, and validated automation into potential business opportunities.**

The project should remain flexible, reproducible, observable, and inexpensive enough to support continuous experimentation.
