# Experiment 001 — Agent Fundamentals

**Project:** OpenClaw Agents Lab  
**Experiment ID:** 001  
**Status:** Completed — Initial Fundamentals  
**Date:** November 8, 2026  
**Platform:** OpenClaw 2026.11.2  
**Primary Agent:** Nexus 🥷

---

## 1. Objective

Understand how OpenClaw initializes, configures, and operates an AI agent.

The experiment focuses on agent identity, persistent instructions, workspace organization, session management, and basic agent execution.

The objective is to establish foundational knowledge before experimenting with specialized agents, external integrations, and autonomous workflows.

---

## 2. Environment

| Component           | Configuration            |
| ------------------- | ------------------------ |
| Infrastructure      | AWS EC2                  |
| Operating system    | Ubuntu 24.04 LTS         |
| Instance            | `t3.medium`              |
| OpenClaw            | 2026.11.2                |
| Primary agent       | `nexus`                  |
| Agent configuration | Single-agent setup       |
| AI provider         | OpenAI API               |
| Models tested       | GPT-6 Astra, GPT-5.6 Sol |
| Gateway             | Loopback                 |
| Dashboard access    | SSH tunnel               |
| Execution           | Direct host execution    |
| Sandbox             | Disabled                 |

---

## 3. Experiments Performed

### 3.1 Agent Initialization

OpenClaw was installed using its official installation script.

During onboarding, the single-agent option was selected.

The primary agent was assigned the name:

```text
nexus
```

The initial bootstrap interaction was completed through OpenClaw's Terminal User Interface (TUI).

**Result:** A functional agent was created and connected to the configured AI provider.

### 3.2 Agent Identity

Nexus was configured with a custom identity.

**Name:** Nexus  
**Emoji:** 🥷  
**Language:** Spanish by default, English on request.

Communication preferences:

- Friendly, clear, and direct.
- Humor and moderate sarcasm when appropriate.
- Avoid unnecessary praise.
- Correct inaccurate assumptions with explanations.
- Acknowledge uncertainty.
- Prioritize understanding before automation.

**Result:** Nexus reported saving these preferences in its persistent workspace files.

### 3.3 Workspace Inspection

Nexus performed an authorized read-only inspection of its workspace.

The following files were identified:

| File           | Purpose                                       |
| -------------- | --------------------------------------------- |
| `IDENTITY.md`  | Agent name, identity, emoji, and presentation |
| `SOUL.md`      | Personality and behavioral principles         |
| `USER.md`      | User preferences and project context          |
| `AGENTS.md`    | Operational instructions                      |
| `BOOTSTRAP.md` | Initial agent bootstrap                       |

Additional potential memory files:

- `MEMORY.md`
- `memory/`

These memory files were not initially present.

### 3.4 Persistent Configuration

Nexus was authorized to update its identity and preference files.

The agent reported modifying:

```text
/home/ubuntu/.openclaw/workspace/IDENTITY.md
/home/ubuntu/.openclaw/workspace/USER.md
/home/ubuntu/.openclaw/workspace/SOUL.md
/home/ubuntu/.openclaw/workspace/AGENTS.md
```

The updates were intended to preserve compatible OpenClaw defaults while adding the agreed preferences.

Nexus reported:

- Successful file modifications.
- Verification by rereading the files.
- A local Git commit.
- Automatic removal of `BOOTSTRAP.md`.

**Verification limitation:** These results were reported by the agent; the full file diff was not independently reviewed during this experiment.

### 3.5 Autonomy Policy

An initial policy requiring authorization before every tool invocation proved overly restrictive for routine agent operations.

The policy was revised to permit autonomous work within authorized tasks.

Permitted routine activities include:

- Reading workspace files.
- Maintaining internal documentation.
- Updating preferences and memory.
- Researching public information.
- Creating artifacts within the workspace.

Sensitive activities require explicit authorization, including system configuration changes, credential access, software installation, external publication, and financial operations.

The agent must report modified files and their absolute paths.

**Finding:** Behavioral instructions provide guidance but do not enforce operating-system security boundaries.

### 3.6 Terminal and Dashboard Interaction

Nexus was accessed through:

- OpenClaw TUI.
- OpenClaw Web Dashboard.

The Dashboard was connected through an SSH tunnel to the Gateway.

The agent's conversation was accessible through both interfaces.

The Dashboard was also used to switch between AI models.

**Result:** Both interfaces operated successfully with the primary agent.

### 3.7 Model Comparison

The initial configuration used GPT-6 Astra.

GPT-5.6 Sol was subsequently selected for a technical inspection and documentation task.

Sol generated a structured security assessment describing:

- Host execution capabilities.
- Workspace access.
- Linux privileges.
- Administrative access.
- Security limitations.
- Possible future hardening measures.

**Finding:** A less expensive model was capable of producing useful technical documentation for this specific task.

No controlled benchmark comparing both models has been completed.

---

## 4. Key Findings

### Identity Is Persistent Configuration

An agent's identity is not defined exclusively by its conversational context.

OpenClaw uses workspace files to persist identity, preferences, and operational instructions.

### Agent Instructions Are Not Security Controls

Instructions can specify what an agent should or should not do.

However, they do not prevent operations that remain technically permitted by the execution environment.

### Agent Workspaces Are Not Automatically Sandboxes

The workspace defines where the agent works.

Without additional isolation, the agent may access other files permitted to its operating-system user.

### The Agent Can Maintain Its Own Documentation

With appropriate authorization, Nexus demonstrated a workflow for updating its own identity and instruction files and reporting the results.

### Agents May Use Different Models

The primary agent can be configured to use different model providers and models.

Model quality and execution cost should be evaluated according to task requirements.

### Agent Claims Require Verification

Self-reported changes, successful tool execution, and declared capabilities should be distinguished from independently verified evidence.

---

## 5. Limitations

This experiment did not fully investigate:

- Tool permission enforcement.
- Execution approvals.
- Sandbox isolation.
- Multi-agent delegation.
- Persistent memory retrieval.
- Semantic search.
- Browser automation.
- External communication channels.
- Autonomous scheduling.
- Agent-to-agent communication.

These topics are covered by subsequent roadmap phases.

---

## 6. Lessons Learned

1. Onboarding decisions can significantly change the initial agent architecture.
2. Repeating installation and onboarding improves understanding.
3. A single-agent configuration is a useful starting point.
4. Identity, preferences, and operational rules have different responsibilities.
5. Allowing routine workspace operations reduces unnecessary approval cycles.
6. Technical restrictions must be evaluated independently from behavioral instructions.
7. Model selection should account for quality and cost.
8. File modifications should be traceable and reviewable.
9. A disposable laboratory is useful for learning but does not eliminate external security or billing risks.

---

## 7. Completion Criteria

- [x] OpenClaw installed and operational.
- [x] Single-agent onboarding completed.
- [x] Primary agent created.
- [x] Initial agent response verified.
- [x] Workspace structure inspected.
- [x] Identity and preferences configured.
- [x] Persistent instruction files updated according to the agent's report.
- [x] TUI and Dashboard access verified.
- [x] More than one model tested.
- [ ] Independently inspect the identity-file diff and current configuration.

**Conclusion:** The initial fundamentals experiment achieved its primary learning objectives. Independent verification and deeper security assessment remain separate follow-up activities.

---

## 8. Related Documentation

- [Project Overview](../../docs/project-overview.md)
- [Project Roadmap](../../docs/roadmap.md)
- [Installation Guide](../../docs/installation/aws-ec2-openclaw.md)

---

**Experiment 001 — Completed with follow-up verification pending.**
