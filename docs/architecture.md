# OpenClaw Agents Lab

## System Architecture

**Project:** OpenClaw Agents Lab  
**Document:** `docs/architecture.md`  
**Version:** 0.1  
**Last Updated:** November 8, 2026  
**Status:** Current Architecture Documented — Experimental Environment

---

## 1. Purpose

This document describes the current architecture of the OpenClaw Agents Lab, including its infrastructure, runtime components, agent configuration, network access, security boundaries, and external integrations.

It also identifies architectural limitations and potential future improvements.

The document distinguishes between:

- **Implemented:** Components deployed and observed in the laboratory.
- **Verified:** Behavior confirmed through commands, logs, or direct interaction.
- **Reported:** Information provided by an agent or previous inspection that has not been independently revalidated.
- **Planned:** Future architectural changes not yet implemented.

The laboratory is not intended to represent a production-ready OpenClaw deployment.

Its purpose is to support learning, controlled experimentation, and reproducible configuration.

---

## 2. Architecture Overview

The current environment runs OpenClaw directly on an AWS EC2 instance.

Language model inference is handled by an external API provider.

```text
                     Maintainer's Mac
                            |
                    SSH (TCP 22)
                            |
                            v
                 AWS Security Group
                            |
                            v
                  AWS EC2 Instance
                  Ubuntu 24.04 LTS
                            |
          +-----------------+------------------+
          |                 |                  |
       SSH Server      OpenClaw Gateway    Local Filesystem
          |                 |                  |
          |                 +-- Nexus Agent    +-- Agent Workspace
          |                 +-- Sessions       +-- Identity Files
          |                 +-- Tools          +-- Memory Files
          |                 +-- Skills         +-- Configuration
          |                 +-- Automations
          |                 |
          |                 v
          |            OpenAI API
          |
          +-- SSH Tunnel
                   |
                   v
           Mac localhost:18789
                   |
                   v
           OpenClaw Dashboard
```

The Gateway listens on the EC2 loopback interface.

The Dashboard is accessed through an encrypted SSH tunnel from the maintainer's Mac.

No public inbound rule is configured for the OpenClaw Gateway port.

---

## 3. AWS Infrastructure

### 3.1 Cloud Provider

**Provider:** Amazon Web Services  
**Service:** Amazon EC2  
**Region:** `us-east-1`

### 3.2 Instance Configuration

| Component          | Configuration              |
| ------------------ | -------------------------- |
| Operating system   | Ubuntu Server 24.04 LTS    |
| Instance family    | T3                         |
| Instance type      | `t3.medium`                |
| vCPU               | 2                          |
| Memory             | 4 GiB                      |
| Root storage       | 30 GiB EBS gp3             |
| Storage encryption | AWS-managed EBS encryption |
| Swap               | 2 GiB                      |
| Virtualization     | HVM                        |
| Instance metadata  | IMDSv2 required            |
| IAM instance role  | None                       |
| Purchase model     | On-Demand                  |

The instance was selected to execute OpenClaw and its tools, rather than host a large language model.

### 3.3 Storage Layout

The environment uses an encrypted EBS root volume.

Relevant directories include:

```text
/
├── home/
│   └── ubuntu/
│       ├── .openclaw/
│       │   ├── workspace/
│       │   └── [OpenClaw state and configuration]
│       │
│       └── .npm-global/
│           ├── bin/
│           └── lib/
│
└── swapfile
```

The OpenClaw installation is associated with the Linux user `ubuntu`.

The laboratory does not currently use an external database or dedicated persistent storage service.

### 3.4 Swap

A 2 GiB swap file is configured at:

```text
/swapfile
```

The file is registered in `/etc/fstab` to persist across reboots.

Swap provides additional memory pressure tolerance but does not replace physical RAM.

### 3.5 Infrastructure Lifecycle

The EC2 instance is considered disposable.

The environment may be terminated and recreated when necessary for learning or architecture validation.

Reproducibility depends on documentation and configuration that can be safely reconstructed.

Credentials, private keys, and sensitive runtime state must not be stored in the project repository.

---

## 4. Network Architecture

### 4.1 Network Access

The EC2 instance is deployed within an AWS VPC with an automatically assigned public IPv4 address.

The public IP may change after stopping and starting the instance.

### 4.2 Security Group

A dedicated Security Group is used for the laboratory.

**Security Group:** `AnjrotOpenClawLab`

#### Inbound

| Protocol | Port | Source                       | Purpose            |
| -------- | ---- | ---------------------------- | ------------------ |
| TCP      | 22   | Maintainer's public IP `/32` | SSH administration |

No public inbound rule is configured for OpenClaw's Dashboard or Gateway.

#### Outbound

| Protocol | Port | Destination | Purpose                             |
| -------- | ---- | ----------- | ----------------------------------- |
| TCP      | 80   | Internet    | HTTP resources                      |
| TCP      | 443  | Internet    | Package downloads and external APIs |

The Security Group configuration is based on the installation performed by the maintainer.

A full independent AWS network audit has not been conducted.

### 4.3 SSH Administration

The EC2 instance is administered from macOS using an SSH private key.

The administrative account is:

```text
ubuntu
```

SSH key authentication is used instead of password-based login.

### 4.4 Gateway Binding

The OpenClaw Gateway is configured to listen on:

```text
127.0.0.1:18789
```

This prevents direct access to the Gateway through the EC2 public network interface under the observed configuration.

### 4.5 Dashboard Access

The Dashboard is accessed through an SSH local port-forward.

Example:

```bash
ssh -N \
  -L 18789:127.0.0.1:18789 \
  -i ~/.ssh/AnjrotDev.pem \
  ubuntu@EC2_PUBLIC_IP
```

After establishing the tunnel, the Dashboard is available locally:

```text
http://localhost:18789
```

The SSH tunnel must remain active while using this access method.

Closing the tunnel does not stop the Gateway.

### 4.6 Future Remote Access

Potential future experiments include:

- Tailscale.
- More convenient SSH tunnel management.
- Private network access from multiple authorized devices.

No public reverse proxy or internet-exposed Dashboard is currently required.

---

## 5. OpenClaw Runtime Architecture

### 5.1 Installation

OpenClaw was installed directly on Ubuntu using the official installation process.

**Version observed:** `2026.11.2`

**Node.js version observed:** `24.22.2`

The application was installed under the `ubuntu` user's npm global directory.

```text
/home/ubuntu/.npm-global/
```

The executable is located at:

```text
/home/ubuntu/.npm-global/bin/openclaw
```

The npm global binary directory was added to the user's `PATH`.

### 5.2 Gateway

The Gateway is the central runtime component responsible for coordinating agent sessions, tools, integrations, and other OpenClaw services.

The Gateway runs as a persistent systemd user service.

It is not currently containerized.

### 5.3 User Session Persistence

The initial installer enabled service persistence so that the Gateway can continue operating after the SSH session ends.

The exact systemd configuration should be inspected during a future infrastructure experiment.

### 5.4 Interfaces

The environment supports three interaction methods:

| Interface          | Purpose                                     | Status   |
| ------------------ | ------------------------------------------- | -------- |
| OpenClaw CLI       | Administration and diagnostics              | Verified |
| OpenClaw TUI       | Terminal-based agent interaction            | Verified |
| OpenClaw Dashboard | Browser-based interaction and configuration | Verified |

The Dashboard and TUI have been used to interact with the same primary agent.

---

## 6. Agent Architecture

### 6.1 Current Deployment

The current deployment uses a single-agent configuration.

**Agent ID:** `nexus`  
**Identity:** Nexus 🥷  
**Role:** Personal AI assistant and future coordinator

The initial installation used a single-agent onboarding flow.

No additional specialized agents are required for the current environment.

### 6.2 Agent Responsibilities

Nexus is intended to support:

- Technical learning.
- Research.
- Planning.
- Documentation.
- Workspace organization.
- Controlled tool execution.
- Future coordination of specialized agents.

Nexus is not currently responsible for executing commercial automation or production workflows.

### 6.3 Agent Workspace

The observed workspace is:

```text
/home/ubuntu/.openclaw/workspace/
```

Relevant files include:

| File          | Responsibility                        |
| ------------- | ------------------------------------- |
| `IDENTITY.md` | Identity and presentation             |
| `SOUL.md`     | Personality and behavioral principles |
| `USER.md`     | Maintainer preferences and context    |
| `AGENTS.md`   | Agent operational instructions        |
| `MEMORY.md`   | Potential long-term memory            |
| `memory/`     | Potential memory records              |

The initial `BOOTSTRAP.md` file was reported as automatically removed after identity initialization.

### 6.4 Persistent Instructions

Nexus was configured to communicate primarily in Spanish, with English available on request.

Its intended communication style is:

- Clear and direct.
- Friendly and professional.
- Willing to identify and explain errors.
- Honest about uncertainty.
- Capable of moderate humor and sarcasm.
- Focused on learning before automation.

The agent reported saving these preferences in its workspace files.

The full contents and changes have not been independently reviewed.

### 6.5 Agent Autonomy

The current behavioral policy permits Nexus to:

- Read and maintain workspace files.
- Organize internal documentation.
- Maintain preferences and memory.
- Research public information.
- Produce artifacts within authorized tasks.

Sensitive operations require explicit human approval.

Examples include:

- System configuration changes.
- Software installation.
- Credential access.
- External publication.
- Financial operations.
- Changes to production systems.

These are behavioral expectations rather than enforced operating-system restrictions.

---

## 7. AI Model Architecture

### 7.1 Current Provider

**Provider:** OpenAI API

Models tested:

- GPT-6 Astra.
- GPT-5.6 Sol.

Astra was initially used for configuration and more demanding work.

Sol was subsequently tested for technical analysis and documentation.

### 7.2 Model Execution

The Gateway sends model requests to the external API provider.

The EC2 instance does not currently execute a local conversational LLM.

### 7.3 Future Providers

Google Gemini is planned as a secondary provider.

Future experiments will evaluate:

- Model switching.
- Model-specific agent assignments.
- Cost and quality comparisons.
- Tool-calling reliability.
- Model fallback strategies.

### 7.4 API Cost

API consumption is separate from EC2 infrastructure costs.

Costs may include:

- Input tokens.
- Output tokens.
- Cached tokens.
- Embeddings.
- Auxiliary model calls.
- Background agent activity.

Cost management is an explicit part of the laboratory.

---

## 8. Memory Architecture

### 8.1 Current State

OpenClaw supports persistent memory through workspace files and memory-search capabilities.

The laboratory has observed embedding API usage but has not yet fully inspected its memory configuration.

### 8.2 Memory Components

The project will investigate:

- Conversation history.
- Context windows.
- Persistent Markdown memory.
- Memory indexing.
- Semantic retrieval.
- Context compaction.
- Background memory maintenance.

### 8.3 Embeddings

The current API usage dashboard reported calls to an OpenAI embedding model.

The laboratory has not yet performed a controlled memory retrieval comparison.

### 8.4 Future Hybrid Architecture

A future experiment will evaluate local embeddings using Ollama, LM Studio, or llama.cpp.

```text
OpenClaw Gateway
       |
       +-- Conversational Reasoning
       |        |
       |        v
       |    Cloud AI API
       |
       +-- Memory Embeddings
                |
                v
         Local Embedding Model
                |
                v
         Local Memory Index
```

The objective is to compare retrieval quality, operational complexity, privacy, memory usage, and cost.

Local embeddings will not automatically eliminate cloud LLM consumption for reasoning or memory summarization.

---

## 9. Tools and Execution Architecture

### 9.1 Current Execution Mode

The initial sandbox inspection reported:

```text
runtime: direct
mode: off
```

This means the inspected agent session executed tools directly in the host environment without an active sandbox.

### 9.2 Available Capabilities

Reported or observed tool categories include:

- Filesystem operations.
- Command execution.
- Process management.
- Session inspection.
- Subagent creation.
- Browser control.
- Internal administrative tools.

Tool availability does not necessarily mean that every integration is configured or authorized.

### 9.3 Tool Execution Privileges

The Gateway operates under the Linux user `ubuntu`.

A previous agent-assisted inspection reported:

```text
sudo: NOPASSWD ALL
```

This creates a significant risk because direct command execution may inherit the privileges available to that user.

The current deployment is intentionally exploratory and should not be considered hardened.

### 9.4 Execution Approvals

The agent-assisted inspection reported broad command execution settings with limited interactive approval requirements.

Execution approval policies have not yet been experimentally evaluated.

### 9.5 Tool Security

The laboratory distinguishes between:

1. **Behavioral instructions:** What an agent is told to do.
2. **Tool policies:** Which tools an agent can invoke.
3. **Execution approvals:** Which operations require confirmation.
4. **Operating-system permissions:** What the executing process can actually access.
5. **Sandbox isolation:** Where execution occurs and what resources are visible.

These mechanisms are complementary rather than interchangeable.

---

## 10. Current Security Assessment

### 10.1 Initial Audit

The command:

```bash
openclaw security audit --deep
```

reported:

```text
0 critical
1 warning
1 information
```

The warning concerned missing trusted proxy configuration.

The Gateway is currently accessed through loopback and SSH tunneling, without a reverse proxy.

No change to trusted proxies was required for the current access pattern.

### 10.2 Identified Risks

| Risk                          | Severity                      | Current Status                     |
| ----------------------------- | ----------------------------- | ---------------------------------- |
| Direct host command execution | High                          | Accepted for laboratory            |
| Passwordless sudo             | High                          | Not remediated                     |
| Sandbox disabled              | High                          | Intentional                        |
| Broad filesystem access       | High                          | Not technically restricted         |
| External API consumption      | Medium                        | Monitored                          |
| Browser automation            | Medium                        | Enabled; usage not fully evaluated |
| Administrative device access  | Requires review               | Not fully audited                  |
| Public Gateway exposure       | Controlled by loopback and SG | No public port configured          |

Severity levels are laboratory assessments, not official OpenClaw audit classifications.

### 10.3 Security Boundaries

The current effective boundaries are primarily:

- AWS Security Group.
- Linux process identity.
- Gateway loopback binding.
- Gateway authentication.
- SSH authentication.
- Operational discipline.

The workspace is not itself a security boundary.

### 10.4 Security Limitation

The current architecture does not protect the EC2 host against all commands that an authorized agent may execute.

The environment must not be connected to production data, credentials, or infrastructure.

---

## 11. Background Execution

### 11.1 Heartbeat

The initial onboarding configured a heartbeat for Nexus.

Observed schedule:

```text
every 30 minutes
```

It was disabled using:

```bash
openclaw config set agents.defaults.heartbeat.every "0m"
```

### 11.2 Memory Dreaming

A daily memory consolidation job was also configured.

It was disabled using:

```bash
openclaw config set \
  plugins.entries.memory-core.config.dreaming.enabled false
```

### 11.3 Current Policy

Background tasks remain disabled until their purpose, model usage, execution cost, and operational boundaries have been evaluated.

Scheduled automation will be introduced during dedicated experiments.

---

## 12. Proposed Future Architecture

The following architecture is a research target, not an implemented deployment.

```text
AWS EC2 / Ubuntu
       |
       +-- ubuntu
       |     |
       |     +-- Human Administration
       |     +-- SSH Access
       |
       +-- openclaw
             |
             +-- OpenClaw Gateway
             |
             +-- Agent Workspace
             |
             +-- Agent Sessions
             |
             +-- Docker Tool Sandbox
             |       |
             |       +-- Restricted Commands
             |       +-- Controlled Filesystem
             |       +-- Limited Network
             |
             +-- Cloud AI Providers
             |
             +-- Local Embedding Model
```

### Proposed Improvements

- Dedicated non-root Linux user.
- No passwordless sudo for the agent service.
- No membership in privileged groups.
- Docker-based tool sandbox.
- Controlled workspace mounts.
- Restrictive tool policies.
- Execution approvals.
- Secure credential management.
- Systemd service hardening.
- Backup and recovery procedures.
- Cost and operational monitoring.

A dedicated Linux user and Docker sandbox address different risks.

Neither should be treated as a complete security solution by itself.

---

## 13. Potential Business Architecture

A future application may use OpenClaw to coordinate affiliate research and SEO content production.

Potential logical components:

```text
Affiliate Research Workflow
          |
          v
Research Agent
          |
          v
Product / Affiliate Analysis
          |
          v
Human Approval
          |
          v
Content Agent
          |
          v
WordPress Draft
          |
          v
Editorial Review
          |
          v
Publication
          |
          v
SEO Performance Monitoring
```

This architecture has not been implemented.

The first validation website will be `anjrot.com`.

Future customer deployments should use appropriately isolated environments and credentials rather than relying on a single shared personal-assistant Gateway.

---

## 14. Architecture Decisions

| ID     | Decision                                                      | Status               |
| ------ | ------------------------------------------------------------- | -------------------- |
| AD-001 | Use AWS EC2 for the initial laboratory                        | Implemented          |
| AD-002 | Use Ubuntu 24.04 LTS                                          | Implemented          |
| AD-003 | Keep Gateway bound to loopback                                | Implemented          |
| AD-004 | Access Dashboard through SSH tunneling                        | Implemented          |
| AD-005 | Use external AI APIs for reasoning                            | Implemented          |
| AD-006 | Begin with a single primary agent                             | Implemented          |
| AD-007 | Retain direct execution for initial learning                  | Accepted temporarily |
| AD-008 | Defer Docker sandboxing                                       | Deferred             |
| AD-009 | Evaluate dedicated Linux user                                 | Planned              |
| AD-010 | Evaluate local memory embeddings                              | Planned              |
| AD-011 | Disable automatic background tasks during initial exploration | Implemented          |
| AD-012 | Keep production infrastructure outside the laboratory         | Implemented          |

These decisions may be revisited as experiments provide new evidence.

---

## 15. Architecture Validation

The architecture should be considered validated for its current learning purpose when the following are confirmed:

- [x] EC2 instance operational.
- [x] SSH access established.
- [x] Gateway running.
- [x] Dashboard accessible through SSH tunnel.
- [x] Primary agent responds.
- [x] OpenAI API integration operational.
- [x] Workspace inspected.
- [x] Initial security audit completed.
- [x] Execution model inspected.
- [ ] Tool approval enforcement verified.
- [ ] Effective filesystem restrictions evaluated.
- [ ] Browser execution environment inspected.
- [ ] Model cost comparison completed.
- [ ] Local embeddings evaluated.
- [ ] Hardened deployment tested.

---

## 16. Related Documentation

- [Project Overview](project-overview.md)
- [Project Roadmap](roadmap.md)
- [AWS Installation Guide](installation/aws-ec2-openclaw.md)
- [Experiment 001 — Agent Fundamentals](../experiments/001-agent-fundamentals/README.md)

---

## 17. Final Notes

This document captures the architecture observed during the initial OpenClaw laboratory deployment.

It is not a production security certification or a complete infrastructure audit.

The current architecture intentionally prioritizes experimentation and learning over strict isolation.

Future changes should be driven by validated requirements and documented experiments.

**Architecture principle: Understand the system before optimizing or restricting it.**
