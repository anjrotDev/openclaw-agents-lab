# OpenClaw Agents Lab

## Project Roadmap

**Project:** OpenClaw Agents Lab  
**Repository:** `openclaw-agents-lab`  
**Document:** `docs/roadmap.md`  
**Version:** 0.1  
**Last Updated:** November 8, 2026  
**Status:** Active — Learning & Experimentation

---

## 1. Roadmap Purpose

This roadmap defines the learning phases, technical experiments, validation milestones, and potential future business initiatives of the OpenClaw Agents Lab.

The project follows an incremental, experimentation-driven approach.

The roadmap is not a fixed implementation schedule. Experiments may be reordered, repeated, postponed, or discarded based on discoveries made during the laboratory.

**Guiding principle: Learn first, validate second, productize third.**

### Status Definitions

| Status      | Meaning                                                 |
| ----------- | ------------------------------------------------------- |
| Completed   | Experiment executed and primary objective achieved      |
| In Progress | Experiment currently being investigated                 |
| Pending     | Planned but not started                                 |
| Deferred    | Intentionally postponed                                 |
| Future      | Potential initiative outside the current learning scope |

Completion applies to the stated learning objective, not necessarily to production readiness.

---

## 2. Roadmap Overview

| Phase | Description                                  | Status      |
| ----- | -------------------------------------------- | ----------- |
| 001   | Cloud Infrastructure & OpenClaw Installation | Completed   |
| 002   | Agent Identity, Workspace & Configuration    | Completed   |
| 003   | Security, Permissions & Execution Model      | In Progress |
| 004   | Tools, Skills & Agent Capabilities           | Pending     |
| 005   | AI Models, Providers & Cost Optimization     | In Progress |
| 006   | Memory, Context & Local Embeddings           | Pending     |
| 007   | Multi-Agent Architecture & Delegation        | Pending     |
| 008   | Automation, Scheduling & Background Tasks    | Pending     |
| 009   | Interfaces, Channels & Remote Access         | In Progress |
| 010   | Reproducible & Hardened Deployment           | In Progress |
| 011   | Affiliate Research & Content Automation      | Future      |
| 012   | Replication & Productization                 | Future      |

---

# Phase 001 — Cloud Infrastructure & OpenClaw Installation

**Status:** Completed

## Objective

Deploy OpenClaw on an isolated cloud server and establish secure administrative access.

## Completed Activities

- Evaluated AWS EC2, DigitalOcean, and Hostinger VPS.
- Selected AWS EC2 for initial experimentation.
- Created an isolated EC2 instance.
- Selected Ubuntu Server 24.04 LTS.
- Configured a `t3.medium` instance with 2 vCPU and 4 GiB RAM.
- Configured 30 GiB encrypted EBS gp3 storage.
- Created a dedicated Security Group.
- Restricted inbound SSH access to the maintainer's public IP.
- Kept the OpenClaw Gateway port closed to the public internet.
- Configured SSH key authentication.
- Updated Ubuntu packages.
- Configured a 2 GiB swap file.
- Installed OpenClaw using its official installer.
- Verified the Node.js runtime.
- Resolved the npm global `PATH` issue.
- Installed and started the OpenClaw Gateway.
- Established access to the Dashboard through an SSH tunnel.
- Verified communication with the OpenAI API.

## Findings

- OpenClaw can run on a relatively small EC2 instance when model inference is handled by an external API.
- The Gateway can remain accessible exclusively through loopback.
- SSH tunneling provides private Dashboard access without exposing its port.
- The installation process is reproducible.
- Repeating the onboarding process revealed configuration options overlooked during the initial installation.

## Acceptance Criteria

- [x] EC2 instance operational.
- [x] SSH administration functional.
- [x] OpenClaw installed.
- [x] Gateway service running.
- [x] Dashboard accessible privately.
- [x] Initial agent response verified.

## Documentation

`docs/installation/aws-ec2-openclaw.md`

---

# Phase 002 — Agent Identity, Workspace & Configuration

**Status:** Completed

## Objective

Understand how OpenClaw initializes an agent and stores its identity, preferences, and operational instructions.

## Completed Activities

- Explored Quick Start and Custom Setup.
- Identified the onboarding option for a single agent versus a team of specialists.
- Repeated onboarding using the single-agent option.
- Created the primary agent `nexus`.
- Investigated workspace structure.
- Inspected identity and instruction files.
- Configured Nexus's identity and communication preferences.
- Defined initial autonomy and approval policies.
- Allowed Nexus to update its workspace documentation.
- Reviewed the agent-generated change report.

## Files Investigated

| File           | Purpose                                       |
| -------------- | --------------------------------------------- |
| `IDENTITY.md`  | Agent identity, name, emoji, and presentation |
| `SOUL.md`      | Personality and behavioral principles         |
| `USER.md`      | Maintainer preferences and context            |
| `AGENTS.md`    | Operational instructions and working rules    |
| `BOOTSTRAP.md` | Initial identity bootstrap                    |
| `MEMORY.md`    | Potential long-term memory                    |
| `memory/`      | Potential persistent memory records           |

## Findings

- Agent identity can be stored in Markdown files.
- Onboarding can generate different agent structures depending on selected options.
- Instructions stored in conversation context are not equivalent to persistent configuration.
- Workspace files can be maintained directly by the agent.
- The agent reported completing its identity configuration and creating a local Git commit.
- Behavioral instructions do not constitute technical security restrictions.

## Acceptance Criteria

- [x] Single primary agent created.
- [x] Workspace structure inspected.
- [x] Identity configured.
- [x] Communication preferences documented.
- [x] Initial autonomy policy established.
- [x] Agent-generated file changes reported.

## Follow-up Research

- Verify identity presentation across interfaces.
- Understand when workspace instructions are loaded into context.
- Explore instruction precedence and configuration inheritance.
- Inspect workspace Git history and lifecycle.

---

# Phase 003 — Security, Permissions & Execution Model

**Status:** In Progress

## Objective

Understand the security boundaries of OpenClaw, including tool permissions, host execution, sandboxing, and administrative access.

## Completed Activities

- Executed `openclaw security audit --deep`.
- Executed `openclaw sandbox explain --agent nexus`.
- Reviewed the Gateway network binding.
- Identified the absence of sandbox isolation.
- Inspected the default execution model.
- Investigated the Linux account used by the Gateway.
- Identified passwordless sudo privileges.
- Documented current architecture and risks using Nexus.
- Reviewed the distinction between sandbox isolation and tool policies.

## Confirmed or Reported Findings

| Area               | Initial Finding                                             |
| ------------------ | ----------------------------------------------------------- |
| Gateway            | Bound to loopback                                           |
| Dashboard          | Accessed through SSH tunnel                                 |
| Sandbox runtime    | Direct                                                      |
| Sandbox mode       | Off                                                         |
| Execution tools    | Broad permissions                                           |
| Host user          | `ubuntu`                                                    |
| sudo               | Passwordless sudo reported                                  |
| Elevated tools     | Feature enabled; effective authorization depends on context |
| Browser control    | Enabled                                                     |
| External messaging | No configured channels observed                             |
| Webhooks           | Disabled in initial audit                                   |
| Trusted proxies    | Not configured                                              |

Some findings were produced by previous inspections and must be revalidated before security-sensitive changes.

## Experiments

### 003.1 — Tool Permission Inspection

**Status:** In Progress

Investigate:

- Effective tool permissions.
- Tool allowlists and denylists.
- Execution security profiles.
- Approval mechanisms.
- Host command execution.
- Workspace-only file access.
- Agent-specific restrictions.

### 003.2 — Docker Tool Sandbox

**Status:** Pending

Explore Docker sandboxing without moving the entire OpenClaw Gateway into a container.

Topics:

- Sandbox backend.
- Container lifecycle.
- Workspace mounts.
- `none`, `ro`, and `rw` access modes.
- Command execution isolation.
- Network restrictions.
- Filesystem boundaries.
- Sandbox escape and privilege limitations.

The experiment should use disposable data and reversible configuration changes.

### 003.3 — Dedicated Linux User

**Status:** Deferred

Evaluate running OpenClaw under a dedicated non-root Linux user without sudo permissions.

Potential architecture:

```text
Ubuntu Host
├── ubuntu
│   └── Human administration
│
└── openclaw
    ├── OpenClaw Gateway
    ├── Agent Workspace
    └── Agent Runtime
```

The dedicated account must not inherit privileged groups or access to administrative sockets.

A dedicated user reduces host privileges but does not automatically create a complete sandbox.

### 003.4 — Execution Approvals

**Status:** Pending

Investigate:

- Approval policies.
- Interactive approvals.
- Safe command execution.
- Execution scope.
- Host versus sandbox execution.
- Failure and denial behavior.

## Acceptance Criteria

- [x] Initial security audit performed.
- [x] Current execution model identified.
- [x] Main privilege risks documented.
- [ ] Tool approval mechanisms understood.
- [ ] Controlled command execution experiment completed.
- [ ] Sandbox behavior evaluated.
- [ ] Dedicated-user deployment evaluated.

---

# Phase 004 — Tools, Skills & Agent Capabilities

**Status:** Pending

## Objective

Understand how OpenClaw extends agent capabilities and how tools differ from skills.

## Planned Experiments

### 004.1 — Built-in Tools

- Discover available tools.
- Inspect tool schemas.
- Understand tool selection.
- Observe tool execution.
- Analyze execution results and failures.

### 004.2 — Skills

- Explore built-in skills.
- Understand skill instructions.
- Investigate skill loading.
- Review security implications.
- Evaluate third-party skills.
- Create a basic custom skill.

### 004.3 — Filesystem Operations

- Read files.
- Create documentation.
- Modify workspace files.
- Inspect changes.
- Evaluate access restrictions.

### 004.4 — Web Research

- Investigate public information.
- Retrieve web pages.
- Produce source-backed reports.
- Handle conflicting or incomplete information.
- Understand prompt injection risks.

### 004.5 — Browser Automation

- Explore browser control.
- Understand browser profiles.
- Test navigation in an isolated environment.
- Evaluate authenticated-session risks.
- Measure resource usage.

## Acceptance Criteria

- [ ] Tool execution lifecycle understood.
- [ ] Built-in tools inspected.
- [ ] At least one custom skill implemented.
- [ ] Web research workflow verified.
- [ ] Browser automation tested safely.
- [ ] Tool permission boundaries documented.

---

# Phase 005 — AI Models, Providers & Cost Optimization

**Status:** In Progress

## Objective

Understand how OpenClaw integrates multiple model providers and how model selection affects quality, reliability, and cost.

## Completed Activities

- Configured the OpenAI API.
- Tested GPT-6 Astra during onboarding and initial configuration.
- Tested GPT-5.6 Sol for technical analysis and documentation.
- Reviewed API usage metrics.
- Investigated input token consumption.
- Investigated embedding usage.
- Linked eligible ChatGPT Pro promotional API credits to the personal API organization.

## Planned Experiments

### 005.1 — Model Comparison

Compare equivalent tasks using different models.

Evaluation criteria:

- Accuracy.
- Instruction following.
- Tool-calling reliability.
- Quality of technical analysis.
- Latency.
- Token consumption.
- Cost per task.

### 005.2 — Gemini Integration

Configure Google Gemini as a secondary provider.

Evaluate:

- Authentication.
- Available models.
- Tool compatibility.
- Cost.
- Response quality.
- Fallback mechanisms.

### 005.3 — Model Assignment by Responsibility

Investigate using different models for:

- Main coordination.
- Research.
- Document generation.
- Classification.
- Routine operations.
- Complex analysis.

### 005.4 — Cost Monitoring

- Analyze input and output tokens.
- Understand cached token billing.
- Track API costs.
- Measure cost per experiment.
- Control background execution.
- Configure effective spending limits where supported.

## Acceptance Criteria

- [x] OpenAI integration operational.
- [x] Two OpenAI models tested.
- [x] Initial API usage inspected.
- [ ] Gemini integrated.
- [ ] Repeatable comparison methodology defined.
- [ ] Cost per representative task measured.
- [ ] Model selection strategy documented.

---

# Phase 006 — Memory, Context & Local Embeddings

**Status:** Pending

## Objective

Understand persistent memory, context management, semantic search, and hybrid AI architectures.

## Planned Experiments

### 006.1 — Context Management

- Understand context windows.
- Inspect context usage.
- Explore conversation history.
- Investigate context compaction.
- Understand how tools and workspace files affect token usage.

### 006.2 — Persistent Memory

- Explore Markdown memory files.
- Understand memory indexing.
- Test retrieval across sessions.
- Evaluate memory retention.
- Investigate memory consolidation.

### 006.3 — Semantic Memory Search

- Inspect the current memory search configuration.
- Identify the embedding provider.
- Understand vector representations.
- Test semantic versus keyword retrieval.
- Evaluate index creation and updates.

### 006.4 — Local Embeddings

**Experiment:** Hybrid AI Architecture — Local Memory Embeddings

Evaluate:

- Ollama.
- LM Studio.
- llama.cpp.
- Small embedding models.
- Local indexing.
- Memory retrieval quality.
- CPU and RAM usage.
- API cost reduction.

The experiment should not require replacing the main conversational model.

### 006.5 — Memory Automation

- Revisit Memory Dreaming.
- Understand scheduled memory maintenance.
- Evaluate model consumption.
- Analyze memory quality and retention.

## Acceptance Criteria

- [ ] Memory architecture understood.
- [ ] Persistent memory tested.
- [ ] Semantic search verified.
- [ ] Local embedding model configured.
- [ ] Retrieval quality evaluated.
- [ ] Resource consumption measured.
- [ ] Hybrid architecture documented.

---

# Phase 007 — Multi-Agent Architecture & Delegation

**Status:** Pending

## Objective

Learn how OpenClaw creates, configures, and coordinates multiple specialized agents.

## Planned Experiments

### 007.1 — Agent Creation

- Create a secondary agent.
- Configure identity.
- Assign a dedicated workspace.
- Select a model.
- Define permissions.

### 007.2 — Agent Delegation

- Delegate a task from Nexus.
- Inspect subagent sessions.
- Retrieve results.
- Handle failures.
- Understand delegation limits.

### 007.3 — Specialized Roles

Candidate agents:

| Agent          | Responsibility                    |
| -------------- | --------------------------------- |
| Nexus          | Primary coordinator               |
| Research Agent | Investigation and source analysis |
| Content Agent  | Editorial drafting                |
| Planning Agent | Task organization                 |
| Analyst Agent  | Structured analysis               |

### 007.4 — Shared and Isolated Context

- Explore session visibility.
- Understand agent-to-agent communication.
- Evaluate shared memory.
- Evaluate isolated workspaces.
- Restrict cross-agent permissions.

## Acceptance Criteria

- [ ] Secondary agent created.
- [ ] Delegation verified.
- [ ] Agent-specific permissions configured.
- [ ] Multiple models tested across agents.
- [ ] Shared versus isolated context documented.

---

# Phase 008 — Automation, Scheduling & Background Tasks

**Status:** Pending

## Objective

Learn how agents execute recurring work and report results without direct user interaction.

## Initial Findings

The onboarding created scheduled background operations:

- Heartbeat.
- Memory Dreaming Promotion.

Both were disabled during initial experimentation.

## Planned Experiments

### 008.1 — Heartbeat

- Understand heartbeat execution.
- Inspect schedules.
- Configure intervals.
- Evaluate idle behavior.
- Monitor model usage.

### 008.2 — Cron Jobs

- Create a scheduled task.
- Execute a task in an isolated session.
- Review task output.
- Inspect logs.
- Handle execution failures.

### 008.3 — Notifications

- Evaluate notification channels.
- Test safe notifications.
- Avoid unauthorized external communication.

### 008.4 — Autonomous Research

- Schedule a research task.
- Generate a report.
- Store results.
- Request human review.
- Measure execution cost.

## Acceptance Criteria

- [ ] Heartbeat behavior understood.
- [ ] Scheduled task executed successfully.
- [ ] Execution logs reviewed.
- [ ] Notification workflow tested.
- [ ] Cost of background execution measured.

---

# Phase 009 — Interfaces, Channels & Remote Access

**Status:** In Progress

## Objective

Explore different ways to interact with and administer OpenClaw.

## Completed Activities

- Used OpenClaw CLI.
- Used OpenClaw TUI.
- Accessed Dashboard through SSH tunneling.
- Used the same primary agent from terminal and web interfaces.
- Changed the selected model from the Dashboard.

## Planned Experiments

### 009.1 — Dashboard

- Explore configuration pages.
- Inspect agents and sessions.
- Understand usage and activity information.
- Evaluate available administrative controls.

### 009.2 — Telegram

- Configure a bot.
- Restrict authorized users.
- Test communication.
- Evaluate channel permissions.
- Measure operational costs.

### 009.3 — Desktop or Mobile Application

- Verify official application availability.
- Investigate supported platforms.
- Evaluate pairing and authentication.
- Compare capabilities with the Dashboard.

### 009.4 — Private Remote Access

- Compare SSH tunneling and Tailscale.
- Evaluate usability.
- Review security implications.
- Avoid unnecessary public exposure.

## Acceptance Criteria

- [x] CLI and TUI operational.
- [x] Dashboard operational.
- [x] SSH tunnel verified.
- [ ] Telegram evaluated.
- [ ] Application availability verified.
- [ ] Preferred daily interface selected.

---

# Phase 010 — Reproducible & Hardened Deployment

**Status:** In Progress — Research

## Objective

Design, implement, and validate a reproducible OpenClaw deployment with reduced privileges and sandboxed agent execution.

The next installation will evaluate the official `openclaw-ansible` project as its primary deployment reference.

The goal is to understand and adapt the reference implementation rather than develop a custom installation process from scratch.

## Research Completed

- Identified the security limitations of the initial deployment.
- Evaluated the benefits of a dedicated Linux service account.
- Investigated Docker-based tool sandboxing.
- Distinguished Gateway execution from sandboxed tool execution.
- Reviewed the relationship between tool policies, execution approvals, and operating-system permissions.
- Identified the official `openclaw-ansible` deployment project.
- Confirmed conceptual alignment between our proposed architecture and documented OpenClaw deployment patterns.

These findings validate the architectural direction, not the security of a deployed implementation.

## Target Architecture

- AWS EC2 running Ubuntu Server 24.04 LTS.
- Dedicated Linux service account without general administrative privileges.
- Human administration through a separate account.
- OpenClaw Gateway running under the dedicated account.
- Docker-based sandboxing for agent tool execution.
- Sandbox enabled for the primary agent.
- Controlled workspace mounts and filesystem access.
- Restricted tool permissions and elevated execution.
- Gateway bound to loopback with authentication.
- Dashboard accessible through SSH tunneling.
- External AI providers configured with protected credentials.
- Reproducible infrastructure configuration using Ansible.

## Planned Activities

### 010.1 — Ansible Reference Implementation Review

**Status:** Pending

- Inspect the official `openclaw-ansible` repository.
- Understand its playbooks, roles, inventory, and variables.
- Review how it creates and manages Linux users.
- Examine Docker installation and sandbox integration.
- Understand its systemd configuration.
- Review its firewall, access, and credential-management approach.
- Identify differences between the reference architecture and our requirements.

### 010.2 — Deployment Design

**Status:** Pending

- Select the applicable Ansible components.
- Define the target EC2 configuration.
- Confirm the dedicated service-account permissions.
- Define the Docker daemon access model.
- Configure sandbox mode and workspace access.
- Establish tool execution and approval policies.
- Document any intentional differences from the reference implementation.

### 010.3 — Reproducible Installation

**Status:** Pending

- Provision a clean EC2 instance.
- Execute the reviewed Ansible installation process.
- Configure OpenClaw under the dedicated service account.
- Apply the reviewed agent identity templates.
- Establish secure Dashboard access.
- Verify service persistence and basic functionality.

### 010.4 — Security Validation

**Status:** Pending

- Verify the effective Linux identity of the Gateway.
- Confirm the service account has no unnecessary administrative privileges.
- Verify tool execution occurs within the sandbox.
- Test authorized workspace access using synthetic files.
- Verify unauthorized host filesystem access is denied.
- Inspect Docker permissions and sandbox mounts.
- Confirm elevated execution cannot bypass intended restrictions.
- Review tool policies and execution approvals.
- Execute the OpenClaw security audit.
- Confirm Gateway network isolation.

### 010.5 — Documentation & Recovery

**Status:** Pending

- Document the validated installation procedure.
- Record architectural decisions and deviations.
- Document troubleshooting findings.
- Define backup and recovery procedures.
- Verify reproducibility from a clean environment.

## Acceptance Criteria

- [x] Initial security limitations identified.
- [x] Target architecture conceptually validated.
- [x] Official Ansible reference implementation identified.
- [ ] Ansible implementation reviewed.
- [ ] Dedicated Linux user configured and verified.
- [ ] OpenClaw Gateway operational under the dedicated account.
- [ ] Docker tool sandbox operational.
- [ ] Workspace isolation tested.
- [ ] Tool execution restrictions verified.
- [ ] Secure Dashboard access confirmed.
- [ ] Clean installation reproducible.
- [ ] Recovery procedure validated.
- [ ] Deployment documentation reviewed.

## References

- [OpenClaw Ansible](https://github.com/openclaw/openclaw-ansible)
- [OpenClaw Sandbox Documentation](https://docs.openclaw.ai/gateway/sandboxing)
- [OpenClaw Docker Backend](https://docs.openclaw.ai/gateway/sandboxing/docker-backend)

**Current conclusion:** The target architecture is conceptually supported by documented OpenClaw deployment practices. Implementation and effective security validation remain pending.

---

# Phase 011 — Affiliate Research & Content Automation

**Status:** Future

## Objective

Develop and validate an AI-assisted affiliate research and SEO content production workflow.

## Initial Environment

**Website:** anjrot.com

## Planned Capabilities

### Research

- Keyword discovery.
- Search intent analysis.
- Competitor analysis.
- Product and service research.
- Affiliate opportunity identification.

### Content

- Research-backed article planning.
- SEO-oriented editorial drafts.
- Internal linking recommendations.
- Source validation.
- WordPress draft generation.

### Affiliate Integration

- Amazon Associates.
- Impact.com.
- Future Mercado Libre integration.

### Performance Measurement

- Search impressions.
- Organic clicks.
- Search query growth.
- Affiliate link clicks.
- Approved commissions.
- Operational cost per article.

## Publishing Policy

All commercial content requires human editorial approval during validation.

The system must not fabricate product testing, reviews, prices, or claims of personal experience.

## Acceptance Criteria

- [ ] Research workflow implemented.
- [ ] Affiliate opportunities identified.
- [ ] Initial articles produced.
- [ ] WordPress draft workflow tested.
- [ ] SEO measurement established.
- [ ] Early organic performance evaluated.

---

# Phase 012 — Replication & Productization

**Status:** Future

## Objective

Determine whether validated automation workflows can be reproduced and eventually offered to third parties.

## Planned Validation

1. Validate the workflow on an existing website.
2. Measure organic search improvements.
3. Evaluate affiliate monetization.
4. Replicate the process on a new website.
5. Assess repeatability.
6. Evaluate the cost of installation and maintenance.
7. Investigate commercial packaging.

## Potential Commercial Model

An installable, customer-specific AI agent system.

Potential components:

- Preconfigured agents.
- Reusable skills.
- Research workflows.
- Content generation workflows.
- WordPress integrations.
- Installation procedures.
- Monitoring and maintenance documentation.

The project does not currently plan a multi-tenant SaaS.

## Acceptance Criteria

- [ ] Initial website validation completed.
- [ ] Repeatability experiment completed.
- [ ] Operational costs understood.
- [ ] Commercial feasibility evaluated.
- [ ] Productization decision documented.

---

# Current Priorities

As of November 8, 2026, the laboratory should focus on the following learning activities:

| Priority | Experiment                            | Reason                                      |
| -------- | ------------------------------------- | ------------------------------------------- |
| 1        | OpenClaw Ansible reference review     | Understand the official deployment approach |
| 2        | Dedicated user and sandbox deployment | Implement the target architecture           |
| 3        | Security validation                   | Verify actual permissions and isolation     |
| 4        | Tools and skills                      | Explore agent capabilities                  |
| 5        | Model comparison and cost monitoring  | Control API consumption                     |
| 6        | Memory and local embeddings           | Explore hybrid AI architecture              |
| 7        | Multi-agent delegation                | Learn specialized agent coordination        |
| 8        | Scheduled automation                  | Explore autonomous execution                |

These priorities may change as new capabilities or technical findings are discovered.

---

# Roadmap Maintenance Rules

1. Update the roadmap after meaningful experiments or architectural decisions.
2. Do not mark an experiment as completed solely because the agent reports success.
3. Record observed evidence and important limitations.
4. Keep security findings explicit.
5. Separate learning objectives from commercial milestones.
6. Do not treat future ideas as implementation commitments.
7. Keep detailed procedures in dedicated documentation.
8. Avoid storing credentials, tokens, private keys, or sensitive data.
9. Preserve a reproducible record of important configuration changes.
10. Reevaluate priorities based on what the laboratory teaches.

---

## Final Note

This roadmap is a living document.

The objective is not to complete every experiment as quickly as possible.

The objective is to understand the technology deeply enough to build secure, useful, and reproducible AI agent systems.

**Learn → Experiment → Validate → Document → Improve → Repeat.**
