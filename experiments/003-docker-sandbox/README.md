# Experiment 003 — OpenClaw Security, Dedicated User & Docker Sandbox

**Project:** OpenClaw Agents Lab  
**Environment:** AWS EC2 — `openclaw-lab-3`  
**Started:** November 8, 2026  
**Last Updated:** November 9, 2026  
**Status:** In Progress — Ansible Baseline and Gateway Verified; Sandbox Pending

> This is an experiment log, **not** a production deployment guide or security certification. Distinguish observed outputs, operator decisions, agent reports, hypotheses, and untested proposals. Never commit credentials, tokens, private SSH keys, or unredacted OpenClaw configuration.

---

## 1. Objective and Method

Evaluate the official `openclaw-ansible` deployment on a disposable EC2 instance, starting with an unmodified upstream baseline and then testing security controls incrementally.

Areas of interest include:

- Dedicated Linux service identity.
- Effective sudo privileges.
- Docker daemon access.
- Sandboxed agent tool execution.
- Gateway availability and persistence.
- Agent workspace initialization.
- Reproducible deployment configuration.
- Potential upstream improvements.

The agreed methodology is:

**Inspect → Deploy with minimal variables → Verify → Document → Change one control → Retest.**

The upstream scoped sudo grants are intentionally retained for the initial baseline. This does **not** constitute acceptance of those permissions for unattended or production workloads.

Neither behavioral instructions nor a running Docker daemon prove tool containment. Potential upstream issues or pull requests require demonstrated causes and repeatable evidence.

### 1.1 Evidence Status

- **Verified:** Output observed from terminal commands or direct Dashboard access supplied by the operator.
- **Reported:** Nexus described performing an operation; independent verification is still needed.
- **Pending:** Not yet tested or not sufficiently evidenced.
- **Design concern:** Risk inferred from inspected configuration, not necessarily a demonstrated failure.

---

## 2. Infrastructure and Access

| Property                  | Observed configuration    |
| ------------------------- | ------------------------- |
| EC2 name                  | `openclaw-lab-3`          |
| Region                    | `us-east-1`               |
| Instance type             | `t3.medium`               |
| Operating system          | Ubuntu Server 24.04.5 LTS |
| Architecture              | x86_64                    |
| CPU                       | 2 vCPU                    |
| RAM                       | Approximately 3.7 GiB     |
| Storage                   | 30 GiB EBS gp3            |
| Initial available storage | Approximately 24 GiB      |
| Swap                      | 2 GiB                     |
| Instance metadata         | IMDSv2 required           |
| IAM instance role         | None                      |
| Administrator             | `ubuntu`                  |
| Service account           | `nexus`                   |
| Gateway port              | `18789`                   |
| Gateway access            | Loopback + SSH tunnel     |

The instance was created as a disposable environment for controlled experimentation.

### 2.1 Administrative Access

The `ubuntu` Linux account is used for SSH administration and installation management.

Authentication is performed using the EC2 SSH private key stored on the maintainer's Mac.

The Security Group permits inbound TCP port 22 only from the maintainer's current public IPv4 address.

The `nexus` service account has no additional SSH authorized key configured through Ansible.

### 2.2 Finding — Dynamic Public IP and SSH Access

After stopping and starting the instance, SSH initially appeared to hang.

**Cause:** The maintainer's public IP address had changed, while the EC2 Security Group still allowed the previous address.

**Resolution:** Updating the inbound SSH rule to the current public IPv4 restored access immediately.

**Lessons learned:**

- An EC2 instance reporting `Running` does not guarantee SSH connectivity.
- The Security Group independently controls network access.
- The maintainer's public IP can change.
- The instance's own public IP can also change after stop/start when no Elastic IP is assigned.

### 2.3 Initial Network Configuration

Observed:

```text
ens5: 172.31.25.2/20
default via 172.31.16.1 dev ens5
172.31.16.0/20 dev ens5
```

This network information will also be relevant when evaluating Docker address allocation.

---

## 3. Ansible Reference and Preflight

**Upstream repository:** https://github.com/openclaw/openclaw-ansible

**Local clone:** `/home/ubuntu/openclaw-ansible`

| Property              | Value                                                      |
| --------------------- | ---------------------------------------------------------- |
| Inspected commit      | `238f100`                                                  |
| Revision date         | November 4, 2026                                           |
| Commit subject        | `fix(release): trust current maintainer signing key (#87)` |
| Git working tree      | Clean at inspection                                        |
| Installation approach | Local playbook                                             |
| Execution script      | `run-playbook.sh`                                          |

The automatic `install.sh` was not used because it downloads an Ansible collection from the moving `main` branch, potentially installing code different from the revision inspected.

Instead, the local `run-playbook.sh` was selected.

### 3.1 Toolchain

| Component                   | Observed version      |
| --------------------------- | --------------------- |
| Ansible Core                | `2.16.3`              |
| Python                      | `3.12.3`              |
| Git                         | `2.43.0`              |
| Node.js after installation  | `v24.22.2`            |
| pnpm after installation     | `12.15.0`             |
| OpenClaw after installation | `2026.11.2 (b556fdf)` |

The repository requires Ansible Core 2.14 or newer.

### 3.2 Ansible Collections

The repository declares these dependencies in `requirements.yml`:

```yaml
collections:
  - name: community.docker
    version: ">=3.4.0"
  - name: community.general
    version: ">=8.0.0"
  - name: ansible.posix
    version: ">=1.5.0"
```

Installation command:

```bash
cd ~/openclaw-ansible
ansible-galaxy collection install -r requirements.yml
```

**Evidence limitation:** The exact resolved collection versions and the full installation transcript were not preserved.

Future runs should record them.

### 3.3 Configuration Overrides

File:

```text
/home/ubuntu/openclaw-lab-3-vars.yml
```

Contents:

```yaml
---
openclaw_user: nexus
openclaw_home: /home/nexus
nodejs_version: "24.x"
openclaw_install_mode: release
tailscale_enabled: false
openclaw_ssh_keys: []
```

These are **Ansible extra variables**, not operating-system environment variables.

No upstream task or template modifications were intentionally introduced for the initial baseline.

### 3.4 Syntax Validation

Command:

```bash
ansible-playbook playbook.yml \
  -e @/home/ubuntu/openclaw-lab-3-vars.yml \
  --syntax-check
```

Observed output:

```text
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available

playbook: playbook.yml
```

The syntax check completed successfully.

The warnings were consistent with the playbook using:

```yaml
hosts: localhost
connection: local
become: true
```

A successful syntax check does not prove task execution correctness or effective security.

### 3.5 Upstream Files Inspected

```text
playbook.yml
playbooks/install.yml
roles/openclaw/defaults/main.yml
roles/openclaw/tasks/main.yml
roles/openclaw/tasks/user.yml
roles/openclaw/tasks/docker-linux.yml
roles/openclaw/tasks/docker-security.yml
roles/openclaw/tasks/firewall-linux.yml
roles/openclaw/tasks/nodejs.yml
roles/openclaw/tasks/openclaw.yml
roles/openclaw/tasks/openclaw-release.yml
roles/openclaw/templates/daemon.json.j2
roles/openclaw/handlers/main.yml
requirements.yml
run-playbook.sh
install.sh
```

The main role executes the following stages:

1. System tools installation.
2. Optional Tailscale setup.
3. Dedicated service user creation.
4. Docker installation.
5. Firewall and host security configuration.
6. Node.js and pnpm installation.
7. OpenClaw package installation.

**Onboarding and Gateway systemd installation are separate, operator-initiated steps.**

---

## 4. Upstream Security Behavior Observed in Source

### 4.1 Linux User and Sudo

The variables `openclaw_user` and `openclaw_home` allow the service account to be customized.

The Ansible `user.yml` task creates a system account with:

```yaml
system: true
shell: /bin/bash
create_home: true
```

The playbook also creates:

```text
/etc/sudoers.d/nexus
```

This file grants specific `NOPASSWD` permissions covering:

- Selected `systemctl` operations targeting a service named `openclaw`.
- `systemctl daemon-reload`.
- Selected Tailscale commands.
- Selected `journalctl` commands.

These grants are more restrictive than `NOPASSWD: ALL`, but are **not equivalent to zero sudo**.

The Tailscale rules are created even when `tailscale_enabled: false`.

The Gateway subsequently created by onboarding is a systemd user service named `openclaw-gateway.service`. Therefore, the practical usefulness of the playbook's `openclaw` system-service sudo rules remains untested.

**Experimental decision:** Preserve the upstream sudo rules during initial functionality testing. Later, remove or restrict unnecessary privileges and repeat the relevant tests.

A future execution of the original playbook may recreate the sudoers configuration.

### 4.2 Docker Daemon and Sandbox Integration

The installer provisions conventional **rootful Docker CE**, not Rootless Docker.

The main Docker service is:

```text
docker.service
```

The `docker-security.yml` task explicitly removes the OpenClaw service account from the `docker` group.

This is intentional because access to the rootful Docker socket provides effectively host-root-equivalent control.

However, this introduces an unresolved question:

**How can the unprivileged Gateway create Docker sandbox containers without obtaining privileged access to the rootful daemon?**

Docker installation does not automatically enable OpenClaw's agent sandbox.

The laboratory must not solve the problem by blindly:

- Adding `nexus` to the rootful `docker` group.
- Making the Docker socket globally writable.
- Mounting the Docker socket into agent containers.
- Disabling the sandbox to make commands succeed.

Potential solutions to evaluate include Rootless Docker or another supported isolation backend.

Actual tool execution must be verified before claiming the agent is sandboxed.

### 4.3 Firewall and Host Security

The inspected playbook configures:

- UFW firewall.
- Incoming traffic denied by default.
- SSH access allowed on TCP port 22.
- Optional Tailscale UDP access.
- Fail2ban for SSH protection.
- Unattended security updates.
- Docker-related firewall rules.
- Docker log rotation.

Firewall changes can disrupt remote administration if configured incorrectly.

Administrative recovery access should remain available during future changes.

### 4.4 Docker Daemon Configuration

The inspected `daemon.json.j2` template contains:

```json
{
  "iptables": true,
  "ip-forward": true,
  "userland-proxy": false,
  "live-restore": true,
  "ip6tables": false,
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "default-address-pools": [
    {
      "base": "172.17.0.0/12",
      "size": 24
    }
  ]
}
```

### 4.5 Design Concern — Docker Network Overlap

The configured pool:

```text
172.17.0.0/12
```

includes the AWS `172.31.x.x` address space.

The EC2 instance uses `172.31.25.2/20`.

This creates a **potential** overlap between Docker-assigned networks and AWS networking.

No actual collision has been demonstrated.

**Baseline decision:** Preserve the upstream template unchanged and inspect the networks created by Docker before deciding whether a change is necessary.

A possible upstream enhancement would allow the Docker address pool to be configured through Ansible variables.

### 4.6 Package Reproducibility

The release installation task runs:

```bash
pnpm install -g openclaw@latest
```

under the dedicated service account.

The observed version was:

```text
OpenClaw 2026.11.2 (b556fdf)
```

However, `@latest` is not a reproducible version pin.

Later installations could resolve to a different release.

The NodeSource repository task also uses `creates`, so changing `nodejs_version` in Ansible variables may not update an existing NodeSource repository configuration during reruns.

Explicit package versions and recorded dependency manifests are future reproducibility improvements.

---

## 5. Baseline Installation and Verification

### 5.1 Execution

The local installer was executed with the custom variables:

```bash
cd ~/openclaw-ansible

./run-playbook.sh \
  -e @/home/ubuntu/openclaw-lab-3-vars.yml \
  -e openclaw_user=nexus
```

The additional `-e openclaw_user=nexus` also allows the wrapper script to display the correct service-user name in its final instructions.

The installer reported:

```text
INSTALLATION COMPLETE!
```

**Evidence limitation:** The full Ansible `PLAY RECAP` was not preserved.

The success message and subsequent runtime checks demonstrate that the installation completed sufficiently for the observed functionality, but do not constitute a complete task-by-task audit.

### 5.2 Linux Account Verification

Commands:

```bash
id nexus
getent passwd nexus
sudo -l -U nexus
```

Observed:

```text
uid=999(nexus) gid=988(nexus) groups=988(nexus)
nexus:x:999:988:OpenClaw Service User:/home/nexus:/bin/bash
```

The account belonged only to its primary group.

The `sudo -l -U nexus` output confirmed the scoped rules described previously, with no reported `NOPASSWD: ALL`.

Runtime enforcement of agent tool restrictions remains untested.

### 5.3 Docker Verification

Commands:

```bash
sudo systemctl is-active docker
sudo docker info --format '{{json .SecurityOptions}}'
getent group docker
```

Observed:

```text
active
["name=apparmor,profile=default","name=seccomp,profile=builtin","name=cgroupns"]
docker:x:987:
```

The Docker daemon was active.

The `docker` group had no listed members.

These Docker security options do not demonstrate that Nexus's tools are sandboxed.

Rootless Docker and the agent sandbox are not yet configured.

### 5.4 Service Account Environment

The operator entered the service account with:

```bash
sudo su - nexus
```

Checks performed:

```bash
whoami
echo "$HOME"
node -v
pnpm -v
openclaw --version
echo "$XDG_RUNTIME_DIR"
echo "$DBUS_SESSION_BUS_ADDRESS"
loginctl show-user nexus -p Linger
```

| Check       | Observed result               |
| ----------- | ----------------------------- |
| `whoami`    | `nexus`                       |
| `HOME`      | `/home/nexus`                 |
| Node.js     | `v24.22.2`                    |
| pnpm        | `12.15.0`                     |
| OpenClaw    | `2026.11.2 (b556fdf)`         |
| XDG runtime | `/run/user/999`               |
| DBus        | `unix:path=/run/user/999/bus` |
| Lingering   | `Linger=yes`                  |

Ansible also generated a one-time welcome message explaining the next onboarding steps.

**Verified:** The service-account shell environment works and OpenClaw is accessible under the correct Linux identity.

**Pending:** Persistence after SSH logout and instance reboot.

---

## 6. OpenClaw Onboarding

The onboarding was started as `nexus`:

```bash
openclaw onboard --install-daemon
```

### 6.1 Configuration Selected

| Area                          | Selection or observation           |
| ----------------------------- | ---------------------------------- |
| Setup                         | Manual setup                       |
| Gateway                       | Local Gateway                      |
| Gateway runtime               | Node.js                            |
| Gateway port                  | `18789`                            |
| Gateway binding               | Loopback                           |
| Tailscale exposure            | Off                                |
| Gateway secret                | Locally generated/stored plaintext |
| AI provider                   | OpenAI                             |
| Model observed                | `openai/gpt-6-astra`               |
| Live AI completion            | Successful                         |
| Chat channels                 | Skipped                            |
| Memory embeddings             | Skipped                            |
| Web search                    | Skipped                            |
| Optional skill dependencies   | Skipped                            |
| Optional plugin installations | Skipped                            |
| Optional plugin configuration | Skipped                            |
| Agent ID                      | `nexus`                            |

The synthetic AI test returned:

```text
AI access works. Replied in 16.3s.
```

This confirms successful access for that request, not a general performance benchmark.

The onboarding reported configuration and session initialization:

```text
Updated config: ~/.openclaw/openclaw.json
Workspace OK: ~/.openclaw/workspace
Sessions OK: ~/.openclaw/agents/nexus/sessions
```

It also reported creating:

```text
~/.openclaw/openclaw.json.bak
```

### 6.2 Gateway Secret Storage

The onboarding offered:

- Generate/store plaintext secret.
- Use SecretRef.

The default plaintext option was selected for the baseline.

The Gateway configuration and credentials must be reviewed later to verify that sensitive files are adequately protected.

Do not publish the generated secret, API credentials or unredacted configuration.

### 6.3 Codex-Related Plugin

During onboarding, OpenClaw reported installing or using a Codex-related plugin/app-server.

Its exact authentication mechanism, effective capabilities and API billing behavior have not been independently verified.

Installing a plugin does not demonstrate that it is actively used or that requests are being billed.

For this reason, the `Codex Hosted Search` option was not enabled.

The description `key-free` must not be interpreted as proof of free or unlimited usage.

### 6.4 Skills with Missing Dependencies

The onboarding offered these 24 skills:

```text
1password
blogwatcher
blucli
camsnap
eightctl
gemini
gh-issues
gifgrep
github
gog
goplaces
himalaya
mcporter
model-usage
nano-pdf
openai-whisper
openhue
oracle
ordercli
sag
songsee
sonoscli
summarize
xurl
```

**Decision:** Skip dependency installation.

Future candidates include:

- `summarize`
- `openai-whisper`
- `github`
- `model-usage`
- `nano-pdf`

The presence of these skills in the onboarding catalog is not proof that their dependencies are installed.

### 6.5 Optional Plugins Offered for Installation

The onboarding presented the following 26 options:

```text
ACPX Runtime
CUA Computer
Diagnostics OpenTelemetry
Diagnostics Prometheus
Diff Viewer Language Pack
Diffs
FaceTime
Fish Audio Speech
GitHub Copilot agent runtime
Google Meet
Gradium
Inworld
llama.cpp Provider
Lobster
Memory LanceDB
Microsoft Teams meetings
Muse Code
MXC Sandbox Execution
ONNX
OpenShell Sandbox
Slack huddles
Team Reports
Tokenjuice
TypeSafe AI
Voice Call
Zoom meetings
```

**Decision:** Skip optional installation.

Potential future research includes observability, alternate sandbox backends, local inference, memory storage and agent runtimes.

### 6.6 Plugins Offered for Configuration

```text
@openclaw/google-plugin
@openclaw/huggingface-provider
@openclaw/minimax-provider
@openclaw/ollama-provider
Anthropic
Codex
Device Pairing
Geolocation
Linux Node
xAI (Grok)
```

**Decision:** Skip optional plugin configuration.

### 6.7 Web Search Providers Offered

```text
Codex Hosted Search
Brave Search
DuckDuckGo Search (experimental)
Exa Search
Firecrawl Search
Firecrawl Search (Free)
Gemini (Google Search)
Grok (xAI)
Kimi (Moonshot)
MiniMax Search
Ollama Web Search
Parallel Search
Parallel Search (Free)
Perplexity Search
SearXNG Search
Tavily Search
```

**Decision:** Skip web-search setup.

The authentication and billing behavior of Codex Hosted Search remain pending investigation.

All optional capabilities will be revisited during the corresponding roadmap phases, rather than enabled indiscriminately during the security baseline.

---

## 7. Findings and Resolutions

### Finding 003-01 — Unsafe systemd Directory Permissions

**Status:** Resolved  
**Root origin:** Pending investigation  
**Date:** November 9, 2026

#### Observed Behavior

During onboarding, Gateway service installation failed:

```text
SERVICE_DEFINITION_UNKNOWN: [unsafe-permissions]
The service directory (~/.config/systemd/user)
or its nearest existing ancestor is group/world-writable.
```

The internal terminal chat and API test were still functional, but the managed Gateway service had not been installed.

#### Diagnosis

Commands:

```bash
namei -l /home/nexus/.config/systemd/user
```

```bash
stat -c '%A %a %U:%G %n' \
  /home/nexus \
  /home/nexus/.config \
  /home/nexus/.config/systemd \
  /home/nexus/.config/systemd/user
```

Observed:

```text
755 nexus:nexus /home/nexus
775 nexus:nexus /home/nexus/.config
```

The `systemd/user` directory did not yet exist.

The group-write permission on `/home/nexus/.config` caused OpenClaw's service-definition safety check to reject the installation.

#### Remediation

Performed as `nexus`:

```bash
chmod 700 /home/nexus/.config
```

Verification:

```bash
stat -c '%A %a %U:%G %n' /home/nexus/.config
```

Observed:

```text
drwx------ 700 nexus:nexus /home/nexus/.config
```

The service installation was retried:

```bash
openclaw gateway install
```

Observed result:

```text
Installed systemd service:
/home/nexus/.config/systemd/user/openclaw-gateway.service

Gateway service installed.
```

#### Conclusion

The permission-related installation failure was resolved without modifying upstream Ansible tasks.

The specific step that originally created `.config` with mode `0775` has not yet been identified.

A potential upstream improvement would ensure that the service directory structure has permissions accepted by OpenClaw.

**Do not claim an Ansible defect until the origin is independently reproduced.**

---

### Finding 003-02 — Concurrent Gateway State Ownership

**Status:** Resolved  
**Date:** November 9, 2026  
**Component:** Gateway startup / SQLite state ownership

#### Observed Behavior

After the systemd service was installed, the Gateway process appeared active, but connectivity failed:

```text
Runtime: running
Connectivity check: failed
connect ECONNREFUSED 127.0.0.1:18789
```

The port was not listening despite systemd reporting an active process.

#### Log Investigation

Application log:

```text
/tmp/openclaw/openclaw-2026-11-09.log
```

Relevant messages:

```text
waiting for Gateway state ownership held by another OpenClaw process,
up to 300 s
```

After approximately 300 seconds:

```text
OpenClaw state database is busy at
/home/nexus/.openclaw/state/openclaw.sqlite

holder 10834 ... The holder 10834 is alive on this host
```

This revealed that another OpenClaw process was retaining ownership of the application state.

#### Root Cause

Process inspection identified an earlier interactive onboarding process:

```text
PID 10819 — openclaw onboard --install-daemon
PID 10834 — openclaw (child of PID 10819)
```

The child process held state ownership while the systemd-managed Gateway attempted to acquire the same state.

This was a process lifecycle conflict, not a demonstrated Docker or network failure.

#### Remediation

The obsolete onboarding process was terminated gracefully:

```bash
kill -TERM 10819
```

Verification:

```bash
ps -p 10819,10834 -o pid,ppid,stat,etime,cmd
```

Neither process remained active.

Then the Gateway service was restarted:

```bash
systemctl --user restart openclaw-gateway.service
```

No SQLite databases or lock files were manually deleted.

#### Verification

The service reported:

```text
Active: active (running)
Main PID: 12970
```

Socket inspection:

```text
LISTEN 127.0.0.1:18789
LISTEN [::1]:18789
```

Deep status:

```text
Gateway version: 2026.11.2
Runtime: running
Connectivity check: ok
Listening: 127.0.0.1:18789
```

The Gateway became operational under the `nexus` Linux account.

#### Lessons Learned

- An active systemd process does not necessarily mean the Gateway is ready.
- OpenClaw processes sharing the same state directory can block each other.
- An interactive onboarding process may remain active after configuration.
- Logs can reveal state ownership conflicts not visible in basic service status.
- Graceful termination and service restart resolved the problem without reinstalling OpenClaw.

**Potential upstream improvement:** Better reporting or prevention of concurrent state ownership during onboarding and Gateway startup.

No upstream defect has yet been established.

---

### Finding 003-03 — Conversational Agent Bootstrap

**Status:** Conversational bootstrap demonstrated; manual preconfiguration pending validation  
**Date:** November 9, 2026

#### Dashboard Access

After the Gateway recovered, the operator established an SSH tunnel from macOS and accessed the Dashboard successfully.

The interface displayed the main Nexus conversation and workspace files.

This confirms that Dashboard access through the private SSH tunnel worked during the test.

Gateway persistence after SSH logout or EC2 reboot remains a separate validation task.

#### Conversational Identity Setup

Instead of copying the previously prepared templates, the operator decided to let Nexus initialize its identity through conversation.

The initial Spanish-language instructions defined:

- Name: Nexus.
- Emoji: 🥷.
- Role: Personal AI assistant.
- Friendly, professional, direct and curious personality.
- Moderate humor and sarcasm when appropriate.
- Communication in the same language used by the operator.
- Concise answers for simple questions and detailed technical explanations when necessary.
- Explicit honesty and correction of mistaken assumptions.
- Workspace autonomy within authorized tasks.
- Approval requirements for system changes, installations, external communications, automation and commercial activities.
- No automatic avatar generation.

The objective was to observe how OpenClaw transforms conversational instructions into persistent workspace files.

#### Changes Reported by Nexus

Nexus reported:

- Updating `IDENTITY.md`.
- Updating `SOUL.md`.
- Updating `USER.md`.
- Synchronizing the visible identity using `openclaw agents set-identity`.
- Verifying the identity using `openclaw agents list --json`.
- Moving `BOOTSTRAP.md` to a recoverable trash location.
- Not installing plugins or creating automations.
- Not generating an avatar.
- Not expanding privileges.

The operator supplied the resulting file contents.

These support the reported file changes, but command execution details and the bootstrap removal location were not independently verified.

#### File Review

**`IDENTITY.md`**

The file contained:

- Nexus as the name.
- The agreed assistant role.
- The chosen personality.
- Emoji 🥷.
- No generated avatar.

**`SOUL.md`**

Nexus retained most of the default structure and added:

- Authorization boundaries.
- Prohibition on unauthorized autonomous tasks.
- Instructions against fabricated information.
- Prompt-injection awareness.
- The agreed communication style.

**`USER.md`**

Nexus created dated directives using the expected metadata format:

```md
<!-- observed: 2026-11-09 | status: active -->
```

However, two quality issues were observed:

- The heading `# USER.md - User Model` was duplicated.
- Instructions mixed English imperative keywords with Spanish text, such as `Always llama...` and `Never inventes...`.

These issues do not necessarily prevent the model from interpreting the instructions, but they demonstrate the need for review of generated configuration.

**`AGENTS.md`**

The file apparently retained its default content.

The original instructions included proactive automation, scheduled checks, external outreach and autonomous Git operations.

Some of these expectations may conflict with the more restrictive instructions in `SOUL.md` and `USER.md`.

This is an instruction-consistency concern, not evidence that the agent has executed unauthorized activities.

#### Workspace Instruction Reloading

Nexus explained that manually editing workspace files should affect behavior when OpenClaw loads their updated context, commonly during a subsequent turn or a new session.

It also clarified:

- `AGENTS.md` defines operational conventions.
- `SOUL.md` defines personality and behavioral principles.
- `USER.md` stores stable user preferences.
- `IDENTITY.md` contains visible identity metadata.
- `openclaw agents set-identity` synchronizes identity information; it does not generally reload all workspace instructions.

This explanation is consistent with the roles of the files, but the exact timing of instruction reloads has not yet been independently tested.

A future experiment should modify a harmless instruction, start a fresh session and verify that the agent incorporates it.

---

#### Manual Workspace Preconfiguration

**Status:** Documented behavior — Pending clean-install validation

The inspected `BOOTSTRAP.md` and Nexus's explanation support a second initialization approach.

Instead of completing identity through conversation, an operator could prepare the workspace files manually before the first agent interaction.

Relevant files:

- `AGENTS.md` — Operational instructions and conventions.
- `IDENTITY.md` — Name, emoji and identity metadata.
- `SOUL.md` — Personality and behavioral principles.
- `USER.md` — Stable preferences.
- `MEMORY.md` — Optional persistent memory.

If the files contain valid configuration and `BOOTSTRAP.md` is absent, the agent is expected to begin the first conversation using the preconfigured workspace instructions rather than initiating the default identity bootstrap.

This approach has not yet been tested in a clean laboratory deployment.

##### Platform Onboarding vs. Agent Bootstrap

These processes are separate.

**Platform onboarding** configures:

- Gateway.
- Model providers.
- API credentials.
- Channels.
- Plugins and integrations.
- Runtime services.

**Agent bootstrap** configures:

- Identity.
- Personality.
- Workspace operating rules.
- User preferences.
- Initial agent context.

Preparing the workspace files does not automatically complete platform onboarding.

##### Visible Identity Synchronization

The visible identity may require synchronization with OpenClaw configuration:

```bash
openclaw agents set-identity \
  --agent nexus \
  --workspace /home/nexus/.openclaw/workspace \
  --from-identity
```

This command updates supported identity metadata.

It is not required merely to make ordinary `AGENTS.md`, `SOUL.md` or `USER.md` instructions available to the agent.

##### Expected Benefits

- Avoid repeating the identity conversation.
- Reduce initial model API consumption.
- Reuse reviewed templates.
- Improve reproducibility.
- Maintain consistent agent configuration between deployments.

##### Pending Validation

A future experiment should verify that:

1. A clean workspace with preconfigured files does not initiate the identity ritual.
2. The first conversation uses the intended instructions.
3. The visible identity is synchronized correctly.
4. Existing workspace files are preserved.
5. Platform onboarding remains independent.
6. New sessions load updated instructions correctly.

**Conclusion:** Conversational bootstrap has been demonstrated. Manual workspace preconfiguration is a promising alternative, but remains unvalidated in this laboratory.

---

## 8. Verification Matrix and Next Tests

| Check                                    | Status              | Evidence or next step                                       |
| ---------------------------------------- | ------------------- | ----------------------------------------------------------- |
| Local Ansible execution                  | Verified            | Installer success                                           |
| Node.js, pnpm and OpenClaw               | Verified            | Runtime version commands                                    |
| Dedicated `nexus` identity               | Verified            | `id`, `getent`, shell checks                                |
| Scoped sudo rules                        | Verified            | `sudo -l -U nexus`                                          |
| No Docker group membership               | Verified            | `id nexus`, `getent group docker`                           |
| Rootful Docker service                   | Verified            | `systemctl is-active docker`                                |
| Gateway service installation             | Verified            | Finding 003-01                                              |
| Gateway listener and connectivity        | Verified            | Finding 003-02                                              |
| Dashboard access through SSH             | Observed            | Dashboard opened successfully                               |
| Conversational workspace changes         | Observed / Reported | Files provided; execution details not independently audited |
| Persistence after SSH logout             | Pending             | Test service from an independent session                    |
| Persistence after EC2 reboot             | Pending             | Reboot and verify service                                   |
| Gateway token and state file permissions | Pending             | Inspect metadata without exposing secrets                   |
| Tool permissions and approvals           | Pending             | Effective policy inspection                                 |
| Rootful Docker socket access             | Pending             | Non-mutating permission check                               |
| Docker network overlap                   | Pending             | Inspect actual routes and Docker networks                   |
| Rootless Docker or alternative sandbox   | Pending             | Evaluate secure integration                                 |
| Main-agent sandbox execution             | Pending             | Effective configuration and actual tool test                |
| Workspace read/write boundaries          | Pending             | Synthetic workspace tests                                   |
| Unmounted host-file denial               | Pending             | Synthetic canary tests                                      |
| Elevated and host execution denial       | Pending             | Negative tests                                              |
| Scoped sudo removal                      | Pending             | Controlled experiment                                       |
| Regression after hardening               | Pending             | Repeat baseline tests                                       |
| Backup and recovery                      | Pending             | Controlled validation                                       |
| Reproducible clean installation          | Pending             | Repeat deployment                                           |
| Preconfigured workspace bootstrap        | Pending             | Clean-install experiment                                    |

### 8.1 Next Experimental Activities

The next stage should focus on understanding and validating actual runtime permissions.

Planned work:

1. Confirm Gateway persistence independently of the original SSH session.
2. Inspect sensitive file permissions.
3. Review effective OpenClaw tool policies.
4. Determine how `nexus` can safely interact with a sandbox backend.
5. Evaluate Rootless Docker or another supported sandbox mechanism.
6. Enable sandbox execution for the main agent.
7. Validate containment using harmless synthetic files and commands.
8. Remove unnecessary scoped sudo grants.
9. Repeat functionality and security tests.
10. Record changes and evaluate upstream contributions.

Do not enable broad tool execution merely because the Gateway and Dashboard are operational.

---

## 9. Current Conclusions

The official Ansible installation produced a working OpenClaw environment under a dedicated Linux service account.

Two independent Gateway problems were diagnosed and resolved:

1. Unsafe permissions on `/home/nexus/.config` prevented installation of the systemd user service.
2. An earlier onboarding process retained SQLite state ownership and prevented the Gateway listener from starting.

After applying the respective corrections, OpenClaw reported successful connectivity on loopback.

The operator also accessed the Dashboard through a private SSH tunnel.

Conversational bootstrap was demonstrated using the initial Nexus identity configuration.

The resulting files show that the agent can generate persistent identity and preference instructions from natural-language interaction, although the generated configuration still requires human review.

**What remains unproven:**

The agent has **not yet been demonstrated to execute its tools inside a sandbox**.

The default rootful Docker daemon is intentionally inaccessible through `docker` group membership.

The remaining work includes effective tool restrictions, sudo hardening, secret protection, sandbox integration, persistence and reproducibility.

This deployment must not yet be classified as hardened or production-ready.

### 9.1 Potential Upstream Contributions

Possible improvement candidates include:

- Preparing systemd user-service directories with compatible permissions.
- Improving reporting and handling of concurrent onboarding and Gateway state ownership.
- Making Docker address pools configurable through Ansible variables.
- Improving reproducibility through version pinning.
- Making administrative sudo grants configurable or optional.

These are research candidates, not confirmed upstream defects.

Any future issue or PR should contain reproducible evidence and distinguish upstream behavior from local environment conditions.

---

## 10. References

- [OpenClaw Ansible](https://github.com/openclaw/openclaw-ansible)
- [OpenClaw Gateway Health](https://docs.openclaw.ai/gateway/health)
- [OpenClaw Gateway Troubleshooting](https://docs.openclaw.ai/gateway/troubleshooting)
- [OpenClaw Security](https://docs.openclaw.ai/security)
- [OpenClaw Sandboxing](https://docs.openclaw.ai/gateway/sandboxing)
- [OpenClaw Docker Backend](https://docs.openclaw.ai/gateway/sandboxing/docker-backend)
- [OpenClaw Agent Workspace](https://docs.openclaw.ai/concepts/agent-workspace)
- [OpenClaw Bootstrapping](https://docs.openclaw.ai/start/bootstrapping)

**Documentation rule:** Store sanitized outputs, exact observed versions and explicit uncertainty. Preserve the upstream repository revision and record future patches separately from the original baseline.
