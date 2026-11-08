# OpenClaw Lab — AWS EC2 Installation Guide

**Project:** OpenClaw AI Agents Lab  
**Milestone:** 001 — First Secure Deployment  
**Platform:** AWS EC2  
**Operating System:** Ubuntu Server 24.04 LTS  
**OpenClaw Version Tested:** 2026.11.1  
**Status:** Initial deployment completed — configuration review pending  
**Last Updated:** November 8, 2026

---

# 1. Overview

This document describes the installation process followed to deploy OpenClaw on an AWS EC2 instance.

The objective is to create a secure laboratory for learning how autonomous AI agents operate, communicate, use tools, maintain memory, and coordinate specialized agents.

The installation uses external AI providers instead of hosting language models locally.

## Architecture

```text
MacBook
   |
   | SSH Tunnel (encrypted)
   |
   v
AWS EC2 - Ubuntu 24.04
   |
   | localhost:18789
   |
   v
OpenClaw Gateway
   |
   +-- Web Dashboard
   |
   +-- Chief of Staff
   |      |
   |      +-- Specialized Agents
   |
   +-- Tools / Skills
   |
   +-- Memory
   |
   +-- Automations
   |
   v
OpenAI API / Gemini API (future)
```

**Security principle:** The OpenClaw Gateway must not be exposed directly to the public internet.

---

# 2. AWS Infrastructure

## 2.1 Region

The laboratory was deployed in:

```text
us-east-1
US East (N. Virginia)
```

## 2.2 EC2 Instance

| Setting           | Value                   |
| ----------------- | ----------------------- |
| Name              | `openclaw-lab`          |
| Instance type     | `t3.medium`             |
| vCPU              | 2                       |
| RAM               | 4 GiB                   |
| AMI               | Ubuntu Server 24.04 LTS |
| Architecture      | x86_64                  |
| Purchase option   | On-Demand               |
| Instance metadata | IMDSv2 required         |
| IAM role          | None initially          |

The instance is intended to run the OpenClaw Gateway and its tools.

Language model inference is handled by external APIs.

## 2.3 Storage

| Setting               | Value     |
| --------------------- | --------- |
| Storage type          | EBS gp3   |
| Size                  | 30 GiB    |
| Encryption            | Enabled   |
| KMS key               | `aws/ebs` |
| Delete on termination | Yes       |

## 2.4 SSH Key Pair

Create or select an AWS EC2 key pair.

Configuration used:

```text
Name: AnjrotDev
Format: PEM
```

The private key must be stored securely on the local Mac.

Never upload the private key to the EC2 instance or commit it to a repository.

---

# 3. Security Group

A dedicated Security Group was created for the laboratory.

```text
Name: AnjrotOpenClawLab
```

## 3.1 Inbound Rules

| Type | Protocol | Port | Source                   |
| ---- | -------- | ---- | ------------------------ |
| SSH  | TCP      | 22   | Personal public IP `/32` |

Only the administrator's current public IP should be allowed.

Do not expose the OpenClaw Gateway port `18789`.

## 3.2 Outbound Rules

| Type  | Protocol | Port | Destination |
| ----- | -------- | ---- | ----------- |
| HTTP  | TCP      | 80   | `0.0.0.0/0` |
| HTTPS | TCP      | 443  | `0.0.0.0/0` |

These rules allow downloading packages and connecting to external APIs over standard web ports.

**Note:** Some future OpenClaw integrations may require additional outbound connectivity. Evaluate each requirement before opening more ports.

## 3.3 Security Considerations

- Do not use the default Security Group.
- Do not expose OpenClaw directly to the internet.
- Restrict SSH access to trusted IP addresses.
- Do not assign unnecessary IAM permissions to EC2.
- Keep the laboratory separate from production infrastructure.
- Use dedicated API keys.
- Avoid running agent tools with root privileges.

---

# 4. Connecting to EC2

## 4.1 Prepare the Private Key on macOS

```bash
mkdir -p ~/.ssh

mv ~/Downloads/AnjrotDev.pem ~/.ssh/

chmod 400 ~/.ssh/AnjrotDev.pem
```

Adjust the source path if the private key is stored elsewhere.

Do not overwrite an existing key unintentionally.

## 4.2 Connect via SSH

Replace `EC2_PUBLIC_IP` with the current public IPv4 address of the instance.

```bash
ssh -i ~/.ssh/AnjrotDev.pem ubuntu@EC2_PUBLIC_IP
```

The default username for the Ubuntu EC2 AMI is:

```text
ubuntu
```

On the first connection, verify the SSH host fingerprint through an independent trusted channel before accepting it.

**Important:** The automatically assigned public IPv4 address can change after stopping and starting the instance.

---

# 5. Verify Ubuntu

Once connected:

```bash
cat /etc/os-release
```

Expected:

```text
Ubuntu 24.04 LTS
```

Check memory:

```bash
free -h
```

Check disk space:

```bash
df -h
```

Initial environment observed:

```text
RAM: approximately 3.7 GiB
Available memory: approximately 3.3 GiB
Root filesystem: approximately 29 GiB
Free disk space: approximately 26 GiB
```

---

# 6. Update the Operating System

Update available packages:

```bash
sudo apt update
sudo apt upgrade -y
```

Check whether a reboot is required:

```bash
if [ -f /var/run/reboot-required ]; then
  echo "Reboot required"
else
  echo "No reboot required"
fi
```

If a reboot is required:

```bash
sudo reboot
```

Reconnect using SSH after the instance becomes available.

**Observed during initial installation:** No reboot was required.

---

# 7. Configure Swap

The original EC2 instance had no swap.

A 2 GiB swap file was created as an additional memory safety margin.

## 7.1 Create Swap File

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

## 7.2 Enable Swap After Reboot

Add the following line to `/etc/fstab`:

```text
/swapfile none swap sw 0 0
```

For a new installation, append it only if it does not already exist:

```bash
grep -q '^/swapfile ' /etc/fstab || \
  echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

## 7.3 Verify

```bash
free -h
swapon --show
```

Expected:

```text
Swap: 2.0 GiB
```

Swap does not increase physical memory or improve processing speed.

It provides a fallback when physical RAM becomes scarce.

---

# 8. Install OpenClaw

## 8.1 Official Installer

OpenClaw was installed using its official installation script:

```bash
curl -fsSL --proto '=https' --tlsv1.2 \
  https://openclaw.ai/install.sh | bash
```

For a security-sensitive environment, review the installer before executing it:

```bash
curl -fsSL --proto '=https' --tlsv1.2 \
  https://openclaw.ai/install.sh \
  -o /tmp/openclaw-install.sh

less /tmp/openclaw-install.sh

bash /tmp/openclaw-install.sh
```

Use one installation method, not both.

The installer handles the OpenClaw installation and required Node.js dependencies when necessary.

## 8.2 Versions Observed

```text
OpenClaw: 2026.11.1
Node.js: 24.22.2
Git: 2.43.0
```

## 8.3 Installation User

OpenClaw was installed under:

```text
/home/ubuntu
```

The installation was performed using the regular `ubuntu` user.

Do not run the OpenClaw Gateway as root unless a specific documented requirement makes it necessary.

---

# 9. OpenClaw Onboarding

The installer launched an interactive setup wizard.

The following options were encountered during the initial installation.

## 9.1 Setup Mode

Options:

```text
Quick start
Custom setup
```

Selected:

```text
Custom setup
```

Reason: The laboratory is intended to explore configuration and security options rather than accept all defaults automatically.

## 9.2 Security Warning

The wizard warned that OpenClaw can access files, execute commands, and use tools with real system permissions.

Selected:

```text
Yes
```

This allowed the setup wizard to continue.

Accepting the warning should not be confused with approving unrestricted future agent actions.

## 9.3 Anonymous Telemetry

Selected:

```text
No thanks
```

This avoids opting into optional usage telemetry during the initial laboratory setup.

## 9.4 Chief of Staff

The wizard requested a name for the coordinating agent.

Selected:

```text
nexus
```

**Important discovery after installation:**

The Dashboard displayed the role **Chief of Staff**, and three additional agents appeared to have been created by the onboarding process.

The actual relationship between the configured name `nexus`, the Chief of Staff role, and those additional agents has not yet been verified.

Do not assume this initial configuration represents the final desired multi-agent architecture.

## 9.5 Environment Discovery

Selected:

```text
Ask first
```

This was preferred over automatic full-access discovery.

## 9.6 Credential Discovery

Selected:

```text
No — I'll configure it manually
```

This prevents the setup process from unnecessarily searching the system for existing provider credentials.

## 9.7 AI Provider

Selected:

```text
OpenAI
```

An OpenAI API key was configured securely through the onboarding wizard.

The initial model shown in the Dashboard was:

```text
openai/gpt-6-astra
```

Gemini was intentionally left for a later experiment.

Do not paste API keys into chat logs, documentation or screenshots.

## 9.8 Memory Embeddings

Selected:

```text
No
```

Semantic memory embeddings were postponed until the basic platform installation could be verified.

This does not necessarily disable all memory capabilities.

## 9.9 Gateway

The setup completed the OpenClaw Gateway installation.

Observed Gateway address:

```text
127.0.0.1:18789
```

The installer configured a persistent user-level systemd service and enabled lingering for the service user.

This allows the Gateway to remain active even after the interactive SSH session ends.

---

# 10. OpenClaw CLI PATH Issue

After installation, the command:

```bash
openclaw --version
```

returned:

```text
command not found
```

The installation itself was successful.

The issue was that the global npm binaries directory was not included in the shell's `PATH`.

## 10.1 Diagnose

```bash
node -v
npm prefix -g
command -v openclaw
```

Observed:

```text
Node.js: v24.22.2
npm global prefix: /home/ubuntu/.npm-global
```

The executable was located at:

```text
/home/ubuntu/.npm-global/bin/openclaw
```

## 10.2 Verify Executable

```bash
ls -l ~/.npm-global/bin/
```

```bash
~/.npm-global/bin/openclaw --version
```

Observed:

```text
OpenClaw 2026.11.1 (4ffdd20)
```

## 10.3 Fix PATH

Add the npm global binaries directory to Bash:

```bash
echo 'export PATH="$HOME/.npm-global/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

For a repeatable installation, avoid inserting duplicate lines if the PATH entry already exists.

Verify:

```bash
command -v openclaw
openclaw --version
```

Expected executable path:

```text
/home/ubuntu/.npm-global/bin/openclaw
```

---

# 11. Secure Dashboard Access

The Dashboard was accessed through an SSH tunnel from macOS.

The Gateway must remain bound to the loopback interface on EC2.

## 11.1 Start SSH Tunnel

In a separate terminal on the Mac:

```bash
ssh -N \
  -L 18789:127.0.0.1:18789 \
  -i ~/.ssh/AnjrotDev.pem \
  ubuntu@EC2_PUBLIC_IP
```

The command does not return to the shell while the tunnel remains active.

## 11.2 Open Dashboard

Open:

```text
http://localhost:18789
```

During the initial onboarding, OpenClaw provided a bootstrap URL containing an authentication token.

That URL was opened locally through the SSH tunnel to complete the initial Dashboard connection.

Never publish or share the bootstrap token.

## 11.3 Tunnel Behavior

- The SSH tunnel must remain active while accessing the Dashboard through this method.
- Closing the tunnel does not stop OpenClaw.
- The Gateway continues running on EC2.
- The tunnel can be reopened whenever remote access is required.

Do not open port `18789` in the AWS Security Group.

---

# 12. Initial Functional Test

After connecting to the Dashboard, a first conversation was initiated with the main agent.

Test instruction:

> We are configuring an AI agent laboratory on AWS to learn OpenClaw. Introduce yourself briefly, explain your role as Chief of Staff, and describe your currently available capabilities. Do not execute tools or modify files.

The agent responded successfully.

### Verified

- OpenClaw Gateway was operational.
- Dashboard was accessible through the SSH tunnel.
- OpenAI API integration worked.
- The agent generated a response.

### Not yet verified

- Actual specialist agent configuration.
- Task delegation.
- Tool permissions.
- Web research execution.
- Browser automation.
- Filesystem isolation.
- Agent-specific workspaces.
- Autonomous task execution.

A successful conversational response does not prove that all declared tools and capabilities are configured.

---

# 13. Inspect Initial Automation Jobs

The installation created background automation jobs.

Because the laboratory uses a paid model API, their behavior was reviewed before enabling further experiments.

## 13.1 Inspect Heartbeat Configuration

```bash
openclaw config get agents.defaults.heartbeat
```

Initial observed output:

```json
{
  "agentId": "nexus"
}
```

The heartbeat interval was not explicitly present in that configuration.

## 13.2 List Scheduled Jobs

```bash
openclaw cron list --all
```

Two initial jobs were observed:

| Job                       | Schedule         |
| ------------------------- | ---------------- |
| Heartbeat (nexus)         | Every 30 minutes |
| Memory Dreaming Promotion | Daily at 03:00   |

---

# 14. Disable Heartbeat

An initial attempt to disable the heartbeat through the cron interface failed.

Attempted command:

```bash
openclaw cron disable <HEARTBEAT_JOB_ID>
```

Error:

```text
system-owned monitor jobs cannot be edited by cron clients
```

This showed that the heartbeat was system-managed.

## 14.1 Correct Configuration

```bash
openclaw config set agents.defaults.heartbeat.every "0m"
```

The CLI confirmed that the change would apply without restarting the Gateway.

## 14.2 Verify

```bash
openclaw config get agents.defaults.heartbeat
```

Then:

```bash
openclaw cron list --all
```

Observed result:

```text
Heartbeat (nexus)
Status: disabled
Next: -
```

The heartbeat was successfully disabled.

---

# 15. Disable Memory Dreaming

The second automatically scheduled job was:

```text
Memory Dreaming Promotion
```

Its job declaration was:

```text
memory-core:memory-dreaming-promotion
```

The job was originally scheduled daily at 03:00.

## 15.1 Inspect Job

The initial installation used:

```bash
openclaw cron show <MEMORY_DREAMING_JOB_ID>
```

The job was enabled and had not executed yet.

## 15.2 Disable Dreaming

```bash
openclaw config set \
  plugins.entries.memory-core.config.dreaming.enabled false
```

The CLI confirmed that the change would apply without restarting the Gateway.

## 15.3 Verify

```bash
openclaw config get \
  plugins.entries.memory-core.config.dreaming
```

Observed output:

```json
{
  "enabled": false
}
```

Then:

```bash
openclaw cron list --all
```

The Memory Dreaming job no longer appeared in the scheduled jobs list.

The heartbeat remained listed as disabled.

**Result:** Both initial periodic automation mechanisms were disabled during laboratory setup.

This does not disable normal interactive conversations with OpenClaw.

---

# 16. Important Security Practices

The agent platform has access to operational tools and should be treated as privileged software.

## Server

- Keep the Gateway bound to `127.0.0.1`.
- Use SSH tunneling for administrative access.
- Do not expose the Dashboard publicly.
- Apply system updates.
- Keep SSH restricted.
- Use a regular Linux account for the Gateway.
- Avoid unnecessary sudo permissions for agent tools.

## Credentials

- Use dedicated API keys.
- Do not store secrets in Git.
- Do not paste keys into documentation.
- Inspect configuration files before sharing them.
- Rotate credentials if accidentally exposed.
- Do not grant unnecessary AWS IAM permissions.

## Agent Execution

- Start with read-only tools.
- Require approval for sensitive actions.
- Do not install arbitrary third-party skills without reviewing them.
- Use isolated test workspaces.
- Review commands before execution where approval controls are available.
- Monitor token consumption and API spending.

## Project Isolation

The OpenClaw laboratory must remain independent from Abidegen.

No production repositories, credentials, databases or infrastructure should be accessible to its agents.

---

# 17. AWS Cost Management

The selected EC2 instance is:

```text
t3.medium
2 vCPU
4 GiB RAM
```

The instance is charged while running.

Additional charges can include:

- EBS storage.
- Public IPv4.
- Data transfer.
- CPU credits under applicable T3 Unlimited usage.
- External AI API consumption.

The instance may be stopped between experiments to reduce compute charges.

However:

- EBS continues to incur storage charges.
- The automatic public IPv4 may change after stop/start.
- OpenClaw will not be available while EC2 is stopped.

AWS Budgets should be configured for billing notifications.

Budget alerts are not equivalent to hard spending limits.

---

# 18. Current Deployment Status

| Component                             | Status    |
| ------------------------------------- | --------- |
| EC2 Ubuntu 24.04                      | Completed |
| Dedicated Security Group              | Completed |
| EBS encryption                        | Completed |
| SSH access                            | Completed |
| Ubuntu updates                        | Completed |
| 2 GiB swap                            | Completed |
| Node.js runtime                       | Completed |
| OpenClaw installation                 | Completed |
| OpenAI API                            | Completed |
| Gateway service                       | Completed |
| SSH tunnel                            | Completed |
| Dashboard access                      | Completed |
| Initial chat test                     | Completed |
| CLI PATH correction                   | Completed |
| Heartbeat disabled                    | Completed |
| Memory Dreaming disabled              | Completed |
| Agent structure inspection            | Pending   |
| Specialist agent configuration review | Pending   |
| Tool permissions review               | Pending   |
| Gemini integration                    | Pending   |
| Memory experiments                    | Pending   |
| Automation experiments                | Pending   |

---

# 19. Known Issues and Findings

## Finding 001 — OpenClaw CLI Missing from PATH

**Cause:** npm global binaries directory was not in the user's PATH.

**Resolution:** Add `~/.npm-global/bin` to `.bashrc`.

**Status:** Resolved.

## Finding 002 — Unexpected Initial Agent Structure

The onboarding appeared to create a Chief of Staff configuration and three additional specialized agents.

The name `nexus` was entered during onboarding, but the Dashboard displayed the primary role as `Chief of Staff`.

**Status:** Investigation required.

Before deleting or reconfiguring agents, inspect:

- OpenClaw's configuration.
- Agent IDs and names.
- Agent workspaces.
- Agent-specific instructions.
- Delegation configuration.
- Tool permissions.

Do not assume those three agents are correctly configured or necessary.

## Finding 003 — Background Automation Enabled by Default

Heartbeat and Memory Dreaming were initially scheduled.

**Resolution:** Both mechanisms were disabled for the initial laboratory.

**Status:** Resolved.

## Finding 004 — Dashboard Workspace Unavailable

The Dashboard initially displayed:

```text
Workspace unavailable
```

The cause has not been determined.

**Status:** Investigation required.

---

# 20. Recommended Next Session

Before enabling additional capabilities or creating new agents:

1. Inspect the existing OpenClaw configuration.
2. Identify all agents created by onboarding.
3. Examine the relationship between Nexus and Chief of Staff.
4. Review current tool permissions and agent workspaces.
5. Decide whether to keep or reset the onboarding configuration.
6. Document the chosen agent architecture.
7. Run a controlled read-only tool experiment.

**Do not reset or delete the current configuration until the existing structure has been inspected and understood.**

---

# 21. Future Laboratory Roadmap

## Stage 1 — Infrastructure

Completed.

## Stage 2 — Agent Fundamentals

- Explore the OpenClaw configuration.
- Understand the Gateway.
- Understand agents and workspaces.
- Explore tools and skills.
- Understand context and memory.

## Stage 3 — Multiple AI Providers

- Configure Google Gemini API.
- Compare OpenAI and Gemini.
- Monitor token consumption.
- Explore model selection and fallback.

## Stage 4 — Specialized Agents

- Research Agent.
- Content Agent.
- Planning Agent.
- Coordinator Agent.

## Stage 5 — Autonomous Workflows

- Scheduled research.
- Periodic reports.
- Notifications.
- Task delegation.
- Human approval workflows.

## Stage 6 — Affiliate Research & Content Automation

Future business use case:

- Product and keyword research.
- Affiliate opportunity analysis.
- SEO content planning.
- WordPress draft generation.
- Human editorial approval.
- Search performance measurement.
- Affiliate commission tracking.

Initial validation website:

```text
anjrot.com
```

The business workflow will be developed only after the laboratory fundamentals have been validated.

---

# 22. Final Notes

This guide documents the first successful deployment of OpenClaw on AWS EC2.

The installation demonstrated that a cloud-hosted agent platform can be accessed privately from macOS using SSH tunneling and connected to an external AI model provider.

The environment is operational, but the multi-agent configuration and tool permissions still require inspection.

The next iteration should prioritize understanding OpenClaw's actual architecture rather than simply completing the installation wizard.

**Project principle:** Learn first, validate second, productize third.
