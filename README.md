# OpenClaw Agents Lab 🦞

**A hands-on laboratory for learning, experimenting with, and building autonomous AI agent systems using OpenClaw.**

This repository documents the journey of exploring AI agents, from cloud infrastructure and agent configuration to tools, memory, security, multi-agent orchestration, and autonomous workflows.

The project is focused on practical learning, reproducible experiments, and identifying useful real-world applications.

> **Learn first. Validate second. Productize third.**

## Overview

OpenClaw Agents Lab is a personal research and development project created to understand how AI agents operate beyond traditional conversational interfaces.

The laboratory explores how agents can:

- Interact with language models through different API providers.
- Execute tools and interact with external systems.
- Maintain identity, context, and persistent memory.
- Research information and produce useful artifacts.
- Coordinate specialized agents and delegate tasks.
- Execute scheduled and autonomous workflows.
- Operate within defined security and permission boundaries.

The environment is intentionally experimental. Infrastructure and configurations may be recreated as part of the learning process.

## Current Architecture

The initial laboratory runs on an AWS EC2 instance using Ubuntu Linux.

```text
                    Maintainer's Mac
                           |
                      SSH Tunnel
                           |
                           v
                  AWS EC2 / Ubuntu
                           |
                    OpenClaw Gateway
                           |
              +------------+------------+
              |            |            |
           Nexus 🥷       Tools        Memory
              |            |            |
              +------------+------------+
                           |
                    External AI APIs
                           |
                    OpenAI / Gemini
```

### Infrastructure

| Component          | Technology                      |
| ------------------ | ------------------------------- |
| Cloud provider     | AWS EC2                         |
| Instance           | `t3.medium` — 2 vCPU, 4 GiB RAM |
| Operating system   | Ubuntu Server 24.04 LTS         |
| Agent platform     | OpenClaw                        |
| Primary agent      | Nexus 🥷                        |
| AI providers       | OpenAI; Google Gemini planned   |
| Administration     | SSH                             |
| Dashboard access   | Private SSH tunnel              |
| Model inference    | External APIs                   |
| Persistent storage | EC2 filesystem                  |

The initial deployment runs OpenClaw directly on Ubuntu without Docker sandboxing.

This setup is intended for learning and is **not a production-hardened deployment**.

See [Project Overview](docs/project-overview.md) for architectural context and project boundaries.

## Project Status

**Active — Learning & Experimentation**

The initial OpenClaw deployment is operational, and the laboratory is currently exploring security, tools, model configuration, memory, and agent execution.

| Area                               | Status      |
| ---------------------------------- | ----------- |
| AWS infrastructure                 | Completed   |
| OpenClaw installation              | Completed   |
| Private Dashboard access           | Completed   |
| Agent identity and configuration   | Completed   |
| Initial security assessment        | Completed   |
| Tools and execution permissions    | In progress |
| Model comparison and cost analysis | In progress |
| Local memory embeddings            | Planned     |
| Docker sandbox experiments         | Planned     |
| Multi-agent orchestration          | Planned     |
| Autonomous workflows               | Planned     |
| Business use case validation       | Future      |

For detailed milestones, dependencies, and experiments, see the [Project Roadmap](docs/roadmap.md).

## Repository Structure

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

Some directories and documents represent planned work and may not exist yet.

### Documentation

- [Experiment 004: Reproducible OpenClaw Installation Guide](experiments/004-reproducible-openclaw-installation/index.html)

| Document                                                        | Purpose                                                                  |
| --------------------------------------------------------------- | ------------------------------------------------------------------------ |
| [Project Overview](docs/project-overview.md)                    | Vision, objectives, scope, principles, and potential business direction  |
| [Project Roadmap](docs/roadmap.md)                              | Learning phases, experiments, progress, and acceptance criteria          |
| [Architecture](docs/architecture.md)                            | Current architecture, components, security boundaries, and future design |
| [AWS Installation Guide](docs/installation/aws-ec2-openclaw.md) | Reproducible EC2 and OpenClaw installation process                       |

## Learning Areas

### Agent Fundamentals

Understanding agent identities, instructions, sessions, workspaces, context, and lifecycle.

### Tools & Skills

Exploring how agents execute tools, interact with files, research information, and extend their capabilities through custom skills.

### Security & Isolation

Evaluating tool permissions, execution approvals, Linux privileges, sandboxing, and access to external systems.

### Memory & Hybrid AI

Investigating persistent memory, semantic search, embeddings, context management, and local models for auxiliary tasks.

### Multi-Agent Systems

Creating specialized agents, assigning different models and permissions, and experimenting with delegation and coordination.

### Automation

Exploring heartbeat, scheduled jobs, autonomous research, notifications, and human approval workflows.

## Experimental Methodology

Each experiment follows a simple process:

1. **Define:** Identify the question or capability to investigate.
2. **Understand:** Review the relevant concepts and documentation.
3. **Implement:** Create the smallest useful experiment.
4. **Observe:** Execute controlled tests and collect evidence.
5. **Evaluate:** Analyze behavior, limitations, security, and cost.
6. **Document:** Record findings and lessons learned.
7. **Repeat:** Improve or recreate the experiment when necessary.

An agent's self-reported success is not considered sufficient evidence of correct execution.

## Security Notice

This repository documents experiments with software capable of executing commands, modifying files, accessing external services, and performing autonomous actions.

The initial environment intentionally uses broad execution permissions to explore OpenClaw's capabilities.

**Do not treat the current configuration as a secure production deployment.**

Important principles:

- Never commit API keys, tokens, SSH private keys, or credentials.
- Keep the OpenClaw Gateway private.
- Use isolated infrastructure for experimentation.
- Review third-party skills and tools before enabling them.
- Require appropriate authorization for sensitive actions.
- Understand the difference between behavioral instructions and technical permission boundaries.
- Avoid connecting experimental agents to production systems.

Future experiments will evaluate dedicated Linux users, Docker tool sandboxing, and more restrictive execution policies.

## AI Providers & Cost Management

The laboratory uses external AI APIs rather than running large language models on the cloud server.

Initial experiments use OpenAI models, with Google Gemini planned as a secondary provider.

Model evaluation considers:

- Accuracy and reliability.
- Tool-calling capabilities.
- Context consumption.
- Latency.
- Input, output, and cached token costs.
- Suitability for specialized responsibilities.

Cloud compute, storage, model usage, and auxiliary services are monitored independently.

The goal is to maintain an affordable environment while prioritizing useful learning outcomes.

## Potential Business Application

### Affiliate Research & Content Automation

A possible future application combines AI-powered commercial research with SEO-oriented content production.

The proposed system would:

1. Research keywords, trends, and commercial opportunities.
2. Analyze search intent and competing content.
3. Identify relevant products and affiliate programs.
4. Prepare research-backed editorial content.
5. Generate WordPress drafts.
6. Support human review and publication.
7. Monitor organic search performance and affiliate results.

The initial validation environment will be **anjrot.com**, potentially using Amazon Associates and Impact.com.

If the workflow demonstrates measurable value, it may be replicated on a new website and eventually packaged as an installable AI agent solution.

**Commercial development is not part of the initial laboratory scope.**

## Project Boundaries

This laboratory is independent of all production business systems.

In particular, **Abidegen and its production infrastructure, repositories, credentials, and data are outside the project's scope**.

The project is not currently building a multi-tenant SaaS or onboarding external customers.

## References

- [OpenClaw Documentation](https://docs.openclaw.ai/)
- [OpenClaw GitHub Repository](https://github.com/openclaw/openclaw)
- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [OpenAI API Documentation](https://platform.openai.com/docs)
- [Google Gemini API Documentation](https://ai.google.dev/gemini-api/docs)

## Maintainer

**Anjrot** — Senior Software Engineer, technical content creator, and independent developer.

- Website: [anjrot.com](https://anjrot.com)
- YouTube: [AnjrotDev](https://www.youtube.com/@anjrotdev)

---

**OpenClaw Agents Lab — Learn. Experiment. Validate. Document. Improve. Repeat.** 🦞
