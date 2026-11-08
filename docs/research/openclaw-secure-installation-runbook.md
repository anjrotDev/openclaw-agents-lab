# OpenClaw Secure Installation Runbook

**Target:** a NEW AWS EC2 instance, Ubuntu Server 24.04 LTS, t3.medium (2 vCPU, 4 GiB RAM), OpenClaw 2026.11.2.

**Prepared:** 2026-11-08. **Status: complete review draft; execution blocked by the explicit gates below.**

This document was completed from evidence already collected during the laboratory investigation. At the owner's request, no further searches were performed to finish it. Creating this document does not authorize installation or infrastructure changes. No procedure below was executed on a new instance; no installation command was tested by drafting this document.

## 1. Purpose and Scope

**Objective:** define a secure, reproducible and testable deployment, while distinguishing proven product behavior from a proposed architecture and unresolved implementation details.

**Actors:** the human operator uses the Mac and AWS console; Linux user `ubuntu` administers the NEW server; Linux user `openclaw` owns and runs the application without sudo or privileged groups. Commands labelled “agent sandbox” must be requested through an actual agent turn, not substituted with a host terminal command.

**Evidence labels**

- **VERIFIED-READ:** command or behavior observed in the existing laboratory or retrieved official documentation/source. This is not an installation test.
- **PROPOSED:** conventional operating-system command or design for the new host. Review it before execution; it has not been tested here.
- **GATE:** version-sensitive syntax, missing evidence or compatibility that must be resolved before that stage. No fabricated command is supplied to fill a gap.

All shell blocks are proposed future operations unless identified as previously observed diagnostics. A shell block is not a script to execute end-to-end. Check the actor, prerequisites and expected result before each block. Never paste the complete document into a shell.

**Out of scope:** migration of the current installation, changes to its users or services, actual provisioning, real-secret penetration tests, public Dashboard access, multiple agents, external channels, scheduled automations and additional integrations.

**Execution release gates:** exact artifact provenance; pinned Node and Docker packages; rootless Docker startup and user mapping; exact 2026.11.2 onboarding/configuration schema; host-execution denial; provider-secret setup; persisted systemd environment; and all acceptance tests. A pending gate means STOP, not “use an insecure fallback.”

**Expected result / verification:** a reviewed procedure with an evidence ledger and acceptance record. **Recovery:** revise the document; do not compensate for missing documentation with administrative experiments on the current server.

## 2. Architecture Overview

**Objective:** separate human administration, Gateway orchestration and sandboxed tool execution.

```text
Mac browser -> localhost:18789 -> SSH tunnel -> EC2 127.0.0.1:18789
Mac SSH ------------------------------------> ubuntu (human administrator)
AWS Security Group: SSH from the approved administrative source only

EC2 Ubuntu 24.04
  ubuntu: administrative account; not the agent's execution identity
  openclaw: dedicated unprivileged Linux account
    systemd user manager with lingering
      rootless Docker daemon -> private user Unix socket
      OpenClaw Gateway -> loopback only, authenticated
        API requests -> OpenAI / Google Gemini
        one main agent -> Docker sandbox, mode all
          persistent authorized workspace: rw
          root filesystem: read-only
          network: none
          no Docker socket, provider keys or host home mount
```

The Gateway remains on the host; enabling tool sandboxing does not move the entire application into Docker. Workspace files remain host-backed and persist across disposable container replacement. [OC-SANDBOX, OC-DOCKER, OC-WORKSPACE]

**Design decision:** rootless Docker, not membership in the rootful `docker` group. The Gateway must reach its rootless daemon, but sandbox processes must not receive that socket or its parent runtime directory. Rootless Docker still grants authority over data accessible to its owning Linux user. A compromised Gateway can therefore affect that user's application state and credentials. This architecture does not isolate the Gateway from itself. [D-ROOTLESS]

**Proposed paths**

| Purpose                                        | Location                                       | Sandbox visibility                              |
| ---------------------------------------------- | ---------------------------------------------- | ----------------------------------------------- |
| Dedicated home                                 | /home/openclaw                                 | Not mounted wholesale                           |
| OpenClaw state and configuration               | /home/openclaw/.openclaw                       | Not mounted wholesale                           |
| Persistent workspace                           | /home/openclaw/.openclaw/workspace             | Only this subtree, rw                           |
| Rootless Docker socket                         | /run/user/DEDICATED_UID/docker.sock            | Never mounted                                   |
| Package binaries/source and image build inputs | Outside workspace                              | Not writable through workspace mount            |
| Provider credentials                           | Supported host-side store outside workspace    | Never mounted or injected into tool environment |
| Backups                                        | Protected administrator-controlled destination | Never mounted                                   |

**Actor:** technical reviewer. **Commands:** none at design stage. **Verification:** inspect actual mounts and process identities after deployment. **Risk:** rw workspace permits deletion and modification of its real files, including identity instructions. **Recovery:** versioned workspace backups, not an assumption that a disposable container protects mounted data.

## 3. Prerequisites

**Objective:** resolve environmental and version assumptions before provisioning. **Actor:** human administrator.

Required: AWS administrative access limited to this deployment; an SSH key managed on macOS; trusted Ubuntu 24.04 amd64 AMI; EBS capacity for Node, application, Docker image layers, logs and backups; API accounts with billing controls; and an independent recovery path.

The inspected local package declares version `2026.11.2` and Node engines `>=24.16.0 <25 || >=26.1.0`. Use a pinned supported Node 24 release, not an assumed Ubuntu package or a floating “latest”. The locally inspected package did not provide a gitHead establishing its upstream build provenance. [OC-LOCAL]

**GATE P1 — reproducibility manifest:** record the approved AMI ID/region, architecture, Node version and checksum, npm package version and integrity, Docker package versions, image source commit, base-image digest and final image ID/digest. Do not claim bit-for-bit reproducibility from a versioned script that still downloads mutable packages.

Proposed checks on the NEW instance:

```bash
# ubuntu; read-only
cat /etc/os-release
uname -m
free -h
df -h /
```

**Expected:** Ubuntu 24.04, x86_64, planned memory/disk. **Risk:** 4 GiB is the requested starting capacity, not a guarantee for concurrent agents or large builds. **Recovery:** stop, adjust sizing or build images separately; do not remove isolation to reduce resource usage.

## 4. AWS Infrastructure Configuration

**Objective:** provision a non-public Gateway while preserving administrative access. **Actor:** human AWS operator; use the console or separately reviewed infrastructure-as-code. No AWS CLI provisioning command is asserted as verified here.

1. Launch the approved Ubuntu 24.04 amd64 AMI on t3.medium.
2. Encrypt the EBS volume; define retention and snapshot policy explicitly. Do not create a recurring schedule as part of this document's execution without separate authorization.
3. Permit inbound TCP 22 only from the administrator's current public IPv4 /32. If IPv6 is used, define equally narrow IPv6 rules; do not leave ::/0 open accidentally.
4. Do not add inbound rules for 18789, Docker API ports or container ports.
5. Require IMDSv2. Prefer no instance IAM role initially; if a role becomes necessary, scope it narrowly. IMDSv2 alone is not proof that host processes cannot acquire role credentials.
6. Review outbound access: the host needs approved package registries, DNS, time and provider HTTPS. Sandbox network none is separate from host egress. Do not pretend that basic Security Group rules provide domain-aware provider filtering.
7. Record network/VPC/route choices, administrator recovery method and cost controls, including burstable CPU credit behavior.

**Verification:** inspect AWS rules and instance metadata settings in the console; verify no public Gateway rule. Later combine this with host listener inspection and an external connectivity check. **Risks:** a changing home IP can lock out SSH; t3 burst capacity and image storage can add cost. **Recovery:** use an authorized AWS administrative path to repair the source rule; never open SSH globally as the default workaround.

**GATE P2:** AWS settings above are design requirements, not AWS controls inspected on the future instance. Official AWS pages in the references are review targets, not claimed as consulted evidence.

## 5. Ubuntu Initial Preparation

**Objective:** establish a maintained host before service installation. **Actor:** ubuntu, using sudo only for human-approved host preparation.

```bash
# ubuntu; PROPOSED; NEW instance only
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install ca-certificates curl git uidmap dbus-user-session
```

Review the package transaction rather than applying an unattended generic hardening script. Install the approved Node distribution using a verified release and checksum procedure: **GATE P3**, exact acquisition and verification steps not recovered sufficiently to prescribe here. Do not use curl-pipe-shell as a substitute.

After a separately authorized reboot, if required, re-establish SSH and repeat OS/resource checks. Reconnect before proceeding with account changes. Review SSH key authentication and password-login policy without blindly replacing sshd configuration. Keep the existing administrative session open when testing changes.

**Expected:** patched host with required prerequisites and known Node version. **Verification:** `node --version` and `npm --version` after approved Node installation. **Risks:** package updates can require a reboot; shell PATH and service PATH can differ. **Recovery:** use the recorded image/snapshot and administrative session; do not downgrade packages randomly.

## 6. Dedicated Linux User

**Objective:** create a separate service identity without administrative rights. **Actor:** ubuntu.

```bash
# ubuntu; PROPOSED; stop if this account already exists
getent passwd openclaw
# A returned account requires inspection, not recreation or deletion.
# Only after confirming the account is absent:
sudo adduser --disabled-password --gecos '' openclaw
sudo chmod 0700 /home/openclaw
id openclaw
sudo loginctl enable-linger openclaw
loginctl show-user openclaw -p Linger
```

A disabled password is not the same as denying every login mechanism. Do not install an SSH authorized key for the service account by default. Human administration remains through ubuntu. Do not add openclaw to sudo, docker, lxd or equivalent privileged groups. Check existing sudoers rules, not just group membership.

To enter a service-account shell for approved setup:

```bash
# ubuntu; PROPOSED handoff to the dedicated identity
sudo -iu openclaw
# now openclaw
id
printf '%s
' "$HOME"
```

**Expected:** openclaw identity, /home/openclaw home, no privileged groups, lingering enabled. **GATE P4:** verify the systemd user manager and /run/user/UID are available in the chosen session mechanism. A sudo login shell does not automatically prove that the user bus is available.

**Recovery:** retain ubuntu access and stop setup if account ownership or pre-existing data is unexpected. Do not delete a populated account as rollback. Lingering permits user services to remain active after logout and start at boot; prove both in section 17.

## 7. Docker Installation and Security

**Objective:** provide Docker management without giving the service account root-equivalent rootful Docker access. **Actors:** ubuntu installs approved packages; openclaw owns its rootless daemon.

**Selected architecture:** Docker rootless. No rootful daemon socket access, no docker group membership, no chmod 666 on a socket, no TCP Docker API, no socket mount inside agent containers.

**GATE D1 — package installation:** use Docker's official Ubuntu repository procedure with verified signing-key handling and pinned versions. The exact repository and package installation block was not fully recovered in this research; do not substitute an unverified command. Include the supported rootless extras and prerequisites. Check whether the package transaction starts a rootful daemon; do not leave it as an accidental alternate backend. Review any action to stop/disable that daemon separately on the NEW instance. [D-UBUNTU, D-ROOTLESS]

**GATE D2 — Ubuntu 24.04:** verify the package-supplied RootlessKit AppArmor integration and subordinate UID/GID allocation before initialization. Do not disable AppArmor globally or set a host-wide unprivileged-user-namespace workaround merely to make a test pass. RootlessKit host permissions and container AppArmor support are different questions. [D-TROUBLE]

Proposed diagnostic block after package installation:

```bash
# openclaw; read-only
id
command -v docker
command -v dockerd-rootless-setuptool.sh
grep '^openclaw:' /etc/subuid /etc/subgid
printf '%s
' "$XDG_RUNTIME_DIR"
systemctl --user is-system-running
```

**GATE D3 — daemon initialization:** use the rootless setup tool's documented installation flow for the pinned Docker release; its full initialization sequence must be confirmed before execution. Then verify the user service and private socket, rather than assuming the active Docker context is safe.

```bash
# openclaw; PROPOSED checks after successful rootless setup
export XDG_RUNTIME_DIR="/run/user/$(id -u)"
export DOCKER_HOST="unix://$XDG_RUNTIME_DIR/docker.sock"
unset DOCKER_CONTEXT
systemctl --user status docker.service
stat -c '%U %G %a %n' "$XDG_RUNTIME_DIR/docker.sock"
docker info --format '{{json .SecurityOptions}}'
id -nG
```

**Expected:** rootless security option, dedicated-user socket, no privileged groups. OpenClaw documentation states that its Docker CLI uses the Gateway process environment; set only one of DOCKER_HOST or DOCKER_CONTEXT. There is no invented sandbox.docker daemon-selection key. [OC-DOCKER]

**Rootless UID/GID trap:** container UID 0 maps to the daemon's unprivileged host UID. Nonzero container UIDs map into subordinate IDs. Retrieved OpenClaw source auto-selects workspace host UID/GID unless docker.user is explicit; applying that raw number inside rootless Docker may make a private workspace unwritable. [D-MAPPING, OC-USER-SOURCE]

**Proposed resolution, GATE D4:** explicitly evaluate container user 0:0 under the proven rootless daemon, with dropped capabilities, read-only root and no-new-privileges. This is namespaced container root, not host root. Verify file ownership and rw behavior with synthetic files before adopting it. Do not chmod 777 the workspace or change it to subordinate ownership without a reviewed design.

**Recovery:** repair the dedicated user's rootless service/environment; never fall back to the rootful socket or grant sudo. Container/image recreation may discard ephemeral state; workspace data requires its own backup.

## 8. OpenClaw Installation

**Objective:** install the exact application version as openclaw, not root. **Actor:** openclaw, after P1/P3 and Docker gates.

Proposed user-local package installation on the NEW instance:

```bash
# openclaw; PROPOSED, not executed in this laboratory
mkdir -p "$HOME/.local"
npm install --global --prefix "$HOME/.local" openclaw@2026.11.2
export PATH="$HOME/.local/bin:$PATH"
node --version
npm --version
openclaw --version
```

Verify the package's registry provenance and integrity BEFORE approving installation, including package lifecycle scripts. Do not install under ubuntu and then recursively transfer that account's entire application state. Pin a concrete Node binary path in the eventual service environment.

The retrieved version-specific source used commit `b556fdf457b026574bf0f1e7b4914b15f3f179ce` for the Docker-user resolver. **GATE P1 remains:** independently establish the relationship between that source, release v2026.11.2 and the published npm artifact. The local package version string alone does not prove byte-identical provenance.

**Expected:** binaries and application package owned by openclaw; no global root installation. **Verification:** `command -v openclaw`, version checks and ownership inspection. **Risk:** npm lifecycle code runs with the installer's privileges; the service user still owns sensitive state. **Recovery:** retain pinned artifacts and manifests; do not overwrite a live installation without an approved rollback plan.

## 9. Gateway and systemd Configuration

**Objective:** a persistent user service with a private rootless Docker connection and loopback-only authenticated Gateway. **Actors:** openclaw configures application/user service; ubuntu supplies only necessary host/user-manager preparation.

**GATE G1:** confirm the 2026.11.2 single-agent onboarding command and service-install flags. Exact setup syntax was not recovered sufficiently to prescribe a runnable command. Use the documented service generator instead of inventing an ExecStart or copying a unit from another version. Do not start normal agent conversations until section 12 controls are effective.

Required effective configuration:

- Exactly one main agent and a workspace under /home/openclaw/.openclaw/workspace.
- Gateway bound to 127.0.0.1:18789 with authentication enabled.
- A systemd USER service running under openclaw; persistent after logout and boot.
- A private runtime environment with the absolute application/Node/Docker executable locations and rootless daemon endpoint.
- No rootful Docker context inherited from the administrator.

Illustrative systemd user-service drop-in, **PROPOSED; apply only after the actual generated unit is identified**:

```ini
[Service]
Environment=DOCKER_HOST=unix://%t/docker.sock
UnsetEnvironment=DOCKER_CONTEXT
```

Here %t must resolve to the dedicated user's runtime directory. Verify the final environment without dumping API keys or tokens. Shell exports do not configure an already-running systemd service. PATH must include the actual pinned Node and Docker locations; do not guess a path in a copied unit.

Do not apply a generic systemd hardening block to both Gateway and rootless Docker: namespace and helper requirements differ. Restrictions such as NoNewPrivileges/ProtectSystem need compatibility tests and may break rootless setup if indiscriminately placed on the daemon.

**VERIFIED-READ diagnostics available in the laboratory:**

```bash
# openclaw; diagnostic commands after installation
openclaw status
openclaw gateway status --deep
```

**Expected:** dedicated identity, loopback listener, authenticated Gateway and reachable rootless daemon. **Recovery:** restore the reviewed user-service drop-in/configuration and restart only the new host's affected service after approval. Preserve SSH access. Never compensate by binding the Gateway publicly.

## 10. Agent Workspace Initialization

**Objective:** initialize one main agent with persistent, limited writable storage. **Actor:** openclaw.

```bash
# openclaw; PROPOSED on a new dedicated home
umask 077
mkdir -p "$HOME/.openclaw/workspace"
chmod 0700 "$HOME/.openclaw" "$HOME/.openclaw/workspace"
```

**GATE W1:** use verified 2026.11.2 onboarding to initialize the single agent; record its actual agent ID instead of assuming the ID is main or nexus. Do not create a second agent just to assign the display name Nexus.

The documented bootstrap seeds AGENTS.md, SOUL.md, IDENTITY.md, USER.md and BOOTSTRAP.md. Customizing identity/preferences can cause bootstrap cleanup automatically. Review that behavior before replacing templates; do not run bootstrap recommendations as permission to install plugins. No avatar generation is required for this baseline. [OC-BOOTSTRAP]

Keep configuration, provider auth, logs, sessions databases, SSH keys, package installation and backups outside the mounted workspace. Do not mount /home/openclaw or the entire .openclaw tree. Review symlinks and nested mounts rather than considering a path prefix sufficient proof of confinement.

**Verification:** workspace ownership, private parent directories, one agent in the agent listing, actual sandbox mount inspection later. **Risk:** writable identity files allow persistent instruction changes; this is deliberate user-authorized autonomy, not immutable policy. **Recovery:** restore reviewed files individually from backup without restoring secrets into the workspace.

## 11. Docker Sandbox Configuration

**Objective:** sandbox normal tool execution for the MAIN agent, not only child sessions. **Actor:** openclaw during human-controlled setup, not an autonomous agent editing its own restrictions.

The following JSON fragment uses fields observed in the retrieved official sandbox documentation; it is a fragment to merge, not a replacement for the complete configuration. Validate against the installed 2026.11.2 schema before use. [OC-SANDBOX, OC-DOCKER]

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "all",
        "backend": "docker",
        "scope": "session",
        "workspaceAccess": "rw",
        "docker": {
          "image": "openclaw-sandbox:REVIEWED-LOCAL-TAG",
          "user": "0:0",
          "readOnlyRoot": true,
          "tmpfs": ["/tmp", "/var/tmp", "/run"],
          "network": "none",
          "capDrop": ["ALL"]
        }
      }
    }
  }
}
```

REVIEWED-LOCAL-TAG is deliberately a placeholder: do not paste this fragment into production unchanged. The user 0:0 decision is conditional on rootless verification in D4. Scope session keeps runtime environments separated; it does not create separate copies of a shared rw workspace.

**Modes:** mode all is necessary for this main-agent requirement; non-main can leave the main session unsandboxed. Tool policy remains a separate layer. [OC-SANDBOX]

| Workspace option | Meaning                                                                                                            |
| ---------------- | ------------------------------------------------------------------------------------------------------------------ |
| none             | A separate sandbox workspace, not access to the main workspace. It does not mean “all filesystem access disabled.” |
| ro               | Main workspace accessible read-only; scratch paths can still be writable.                                          |
| rw               | Authorized main workspace is writable and host-persistent. Changes affect the real host-backed files.              |

**GATE I1 — image/source discrepancy:** retrieved current Docker-backend documentation names openclaw-sandbox:trixie-slim, while the locally inspected 2026.11.2 distribution contains a special missing-image path for openclaw-sandbox:bookworm-slim and refers to scripts/sandbox-setup.sh. This does not establish which image should be selected for every build. Resolve the exact version's default, Dockerfile and build context before building. Do not blindly mix current docs with the installed bundle or substitute bare Debian. The inspected error path explicitly requires Python helpers for write/edit operations.

Base tool, common-tool and browser-image variants must be checked against the chosen source revision. The baseline needs only the tool image; browser sandbox and additional integrations are out of scope. Image-build scripts were not found at the inspected npm package root; do not assume npm ships all repository build inputs. [OC-IMAGES]

Build/pull into the SAME rootless daemon used by the Gateway. Pin the base image digest and dependencies for stronger reproducibility; record the resulting image ID. No external bind mounts, host networking, privileged mode, Docker socket, provider secrets or dangerous mount-validation overrides.

**VERIFIED-READ:**

```bash
# openclaw; use the actual agent ID
openclaw sandbox explain --agent ACTUAL_AGENT_ID
openclaw sandbox list
```

**Expected:** active Docker sandbox policy for main agent; rw workspace only, none network, read-only root, dropped capabilities and no-new-privileges. **Recovery:** retain sandbox all; fix image/permission/environment issues or stop the agent. Do not turn sandbox off as a repair shortcut. Scoped recreation discards ephemeral container state and must be reviewed before use.

## 12. Tool Permissions and Execution Approvals

**Objective:** eliminate alternate host-execution and administration routes while retaining workspace work. **Actor:** human operator configuring openclaw before normal agent use.

Verified documentation exposes this elevated-tool control:

```json
{
  "tools": {
    "elevated": {
      "enabled": false
    }
  }
}
```

Elevated execution is an explicit sandbox escape route; disabling it is necessary but is not proof that all other host paths are blocked. It also does not remove Linux sudo rights from an account that has them. [OC-POLICY]

**GATE T1 — exact tool policy:** validate the 2026.11.2 schema and runtime tool names, then produce a minimal allowlist. Intended permitted capabilities: sandbox-backed file read/write/edit, sandbox exec/process and conversation response. Deny unneeded Gateway/config administration, node/desktop control, host execution, package/plugin management, external messaging, recurring automation and cross-agent administration. Do not invent a list of tool identifiers or assume an unavailable tool name is a working deny rule.

**GATE T2 — execution routes:** explicitly bind normal execution to the sandbox using version-verified policy. A request for host/gateway/node execution and elevated execution must fail. Saved previews may omit session overrides; check the live session and actual tool behavior. Do not grant broad shell/interpreter allowlists for host execution.

**GATE T3 — approvals:** independently set conservative host-execution approvals (deny unless explicitly authorized, with fail-closed fallback), even though normal work must remain sandboxed. The exact configuration-writing command/schema is pending. Approval settings and tool availability are distinct. A fallback deny with ask off and full security is not an ordinary approval barrier.

Diagnostics observed in the laboratory:

```bash
# openclaw; inspection, not policy setters
openclaw exec-policy show
openclaw approvals get --gateway
# Substitute the actual session key; preview is not live execution proof.
openclaw exec-policy show --session ACTUAL_SESSION_KEY
```

A workspace-only file-tool restriction is useful defense in depth, but cannot substitute for shell confinement or OS permissions. Sandbox network none does not automatically disable host-side web tools or provider calls. No channels/plugins/schedules should be enabled by onboarding defaults without review.

**Expected / verification:** synthetic host and elevated requests are denied while ordinary sandbox file work succeeds. **Recovery:** stop normal agent use if any host route succeeds; repair policy and retest. Never expose the rootless socket to let the agent “repair Docker.”

## 13. Secure Dashboard Access

**Objective:** reach an authenticated loopback-only Dashboard from macOS. **Actor:** human on Mac.

```bash
# Mac; PROPOSED; replace both explicit placeholders
ssh -N -T \
  -o ExitOnForwardFailure=yes \
  -o ServerAliveInterval=30 \
  -L 127.0.0.1:18789:127.0.0.1:18789 \
  -i "$HOME/.ssh/APPROVED_KEY" \
  ubuntu@EC2_HOST
```

Verify the server host-key fingerprint through a trusted path on first connection. Do not disable host-key checking. Open http://localhost:18789 on the Mac; complete the documented authentication flow without placing a token in shared URLs, screenshots or this runbook. A tunnel is transport, not a replacement for Gateway authentication.

**Expected:** Dashboard works through the tunnel; Gateway is not reachable directly from the Internet. **Verification:** section 16 combines host listeners, AWS rules and external checks. **Risk:** loopback-only forwarding still allows processes on the Mac to reach the local endpoint; retain authentication. **Recovery:** resolve occupied local port or SSH access; do not expose Gateway publicly. If using another local port, change only the local side deliberately.

## 14. AI Provider Configuration

**Objective:** configure OpenAI and Google Gemini API access without exposing credentials to the sandbox. **Actor:** human operator using the supported OpenClaw secure setup surface under the dedicated identity.

**GATE A1:** the exact 2026.11.2 provider authentication commands, provider identifiers, secret-store references and model IDs were not recovered sufficiently to print verified configuration. Complete this gate using that version's schema and official provider docs before execution. No invented API key command or assumed model alias is supplied.

Required workflow:

1. Create separately scoped provider API keys with appropriate account/project budget controls.
2. Use a supported masked prompt or verified secret-reference mechanism; do not type real keys into shell command arguments, chat or the document.
3. Store credentials only in a supported host-side location outside the workspace and image build context, readable only as necessary by the dedicated Gateway identity.
4. Configure one explicit default model and the second provider for a separately controlled test. Automatic fallback is not assumed or required.
5. Confirm neither provider key appears in sandbox mounts or container environment. Gateway-side HTTPS must work while sandbox egress remains disabled.
6. Make one minimal synthetic request per provider after approval of possible API charges. Never use real documents for the first test.

**Expected:** both APIs usable through Gateway orchestration without secrets in tool execution. **Verification:** successful synthetic responses and metadata-only secret checks; never print full environment/config/log contents. **Risks:** dedicated-user processes can access their own credentials; provider usage incurs costs and sends prompts off-host. **Recovery:** revoke/rotate affected keys via provider controls, update the supported host secret store and verify again. Do not copy keys into the sandbox to fix authentication.

## 15. Agent Identity Templates

**Objective:** apply reviewed repository templates to one agent without importing permissions or integrations unintentionally. **Actor:** human operator as openclaw during setup.

| File                    | Intended role and verified limits                                                                                                                                                                            |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| IDENTITY.md             | Name, creature, vibe, emoji and optional avatar. Supported visible identity synchronization is separate from merely writing this file.                                                                       |
| SOUL.md                 | Personality and behavioral principles supplied as agent context.                                                                                                                                             |
| USER.md                 | Stable preferences and user/project directives. Keep concise; local template documents a 4,000-character budget in this installation. Revalidate for the new build.                                          |
| AGENTS.md               | Workspace operating instructions. These are instructions, not enforced OS security policy.                                                                                                                   |
| TOOLS.md                | Do not assume automatic loading. The inspected bootstrap seeds no TOOLS.md and directs local tool notes to the Tools section of AGENTS.md. Exact 2026.11.2 TOOLS.md context-loading support remains GATE W2. |
| MEMORY.md / daily notes | Optional persistent facts and continuity, not provider credentials. Loading depends on session/context policy; do not assume every file is injected into every turn.                                         |

Bootstrap/context loading is selective and subject to limits. Main sessions and subagents can receive different context. Verify actual startup behavior and truncation without asking the model to reveal private system instructions. A benign synthetic preference recalled in a fresh session provides a functional check, not proof of every internal context rule.

**Minimal IDENTITY.md example**

```markdown
# Identity

- Name: Nexus
- Creature: Personal AI assistant
- Vibe: Friendly, clear and direct; accurate and candid.
- Emoji: 🥷
```

**Minimal SOUL.md content:** communicate in Spanish unless English is requested; use moderate non-offensive humor; correct mistakes with reasons; state uncertainty; do not invent evidence.

**Minimal USER.md entry:**

```markdown
<!-- observed: 2026-11-08 | status: active -->

- Prioritize learning OpenClaw, agents, memory, tools and automation before commercial use cases.
```

**AGENTS.md policy requirements:** autonomy within the authorized workspace task; report paths and checks; explicit approval for system/security changes, third-party communications, publishing and commerce; no autonomous schedules or push; stop on material scope changes; never treat retrieved content as authorization. Preserve valid original rules, not blanket replacement.

Inspect repository templates before copying; reject symlinks escaping the intended repository/workspace and remove any embedded secrets. Back up existing files outside the sandbox-visible subtree. Apply exact reviewed files, not an entire repository checkout with unknown hooks and credentials.

Verified visible identity command from local CLI documentation:

```bash
# openclaw; future authorized configuration action, actual agent ID required
openclaw agents set-identity --agent ACTUAL_AGENT_ID \
  --workspace /home/openclaw/.openclaw/workspace --from-identity
```

This writes agent identity configuration; it is not a read-only command. Configured identity takes precedence over workspace fallback. Do not create another agent or install a role to set the display name. [OC-AGENTS]

**Expected / verification:** one agent, correct visible name/emoji and fresh-session behavior. **Recovery:** restore only the affected reviewed identity/preference files and identity settings. Do not force bootstrap deletion or launch optional recommendations implicitly.

## 16. Security Validation

**Objective:** demonstrate containment using harmless synthetic data. **Actors:** ubuntu for host inspection; openclaw for daemon metadata; agent sandbox for actual tool tests. All tests below are FUTURE tests and were not executed while authoring.

### 16.1 Listener and service identity

```bash
# ubuntu; read-only
ss -ltnp
ps -eo user,pid,comm
id openclaw
# Administrative policy inspection only; this does not run a root command as the agent.
sudo -l -U openclaw
```

Expect 127.0.0.1:18789, no 0.0.0.0/[::] Gateway listener, Gateway owner openclaw and no sudo authorization. Identify the real unit/PID and inspect it rather than relying solely on a process name. The administrator's sudo rights are expected and separate.

On the Mac, with SSH tunnel stopped, a narrowly targeted `nc -vz EC2_HOST 18789` check should not connect (use the Mac's supported timeout option if needed). Failure alone is insufficient: correlate with AWS rules and host listener. Then start the tunnel and verify Dashboard operation. Test IPv6 separately if enabled.

### 16.2 Synthetic host-only canary

Prepare a dedicated-user-readable, unmounted canary so a successful denial tests mount isolation rather than only host file permissions:

```bash
# openclaw HOST shell; synthetic data only; retain printed directory for the test
umask 077
CANARY_DIR=$(mktemp -d "$HOME/openclaw-host-canary.XXXXXX")
printf 'SYNTHETIC-NONSECRET-HOST-CANARY
' > "$CANARY_DIR/marker.txt"
printf '%s
' "$CANARY_DIR"
```

Give the agent only the resulting absolute path. Through an ordinary agent tool turn, request reading that exact file and creating a marker in that host-only directory. Both must fail through file tools AND sandbox exec. Do not ask for real config, SSH keys, provider keys or metadata credentials. Afterward, the human verifies the host canary is unchanged and no extra file was created.

### 16.3 Workspace write and rootless mapping

Through a normal agent turn, ask it to execute these commands inside its observed sandbox workspace, not a guessed host path:

```bash
# AGENT SANDBOX; synthetic test, current directory must be the authorized workspace
pwd
id
TEST_DIR=$(mktemp -d ./sandbox-check.XXXXXX)
printf 'SYNTHETIC-WORKSPACE-CHECK
' > "$TEST_DIR/marker.txt"
cat "$TEST_DIR/marker.txt"
printf '%s
' "$TEST_DIR"
```

The human confirms the file exists in the corresponding host workspace and has acceptable ownership. If using the proposed rootless user 0:0, inside id may show root while host ownership remains openclaw. Require daemon rootless proof before interpreting this as acceptable. Retest after scoped container replacement to prove persistence, only after approval to discard ephemeral state.

### 16.4 Container metadata and sockets

Select the exact container from the sandbox listing; never operate on all Docker containers indiscriminately.

```bash
# openclaw HOST; replace SANDBOX_CONTAINER_ID with the inspected ID
# Output only selected metadata, never the full container environment.
docker inspect --format '{{json .Mounts}}' SANDBOX_CONTAINER_ID
docker inspect --format '{{.Config.User}} {{.HostConfig.NetworkMode}} {{.HostConfig.ReadonlyRootfs}} {{.HostConfig.Privileged}}' SANDBOX_CONTAINER_ID
docker inspect --format '{{json .HostConfig.CapDrop}} {{json .HostConfig.SecurityOpt}}' SANDBOX_CONTAINER_ID
```

Expect approved workspace and any explicitly reviewed Gateway-managed read-only resource mounts only; no home/state root, Docker socket, runtime parent directory, credentials or unrelated host directories. Network none, read-only root true, privileged false, capabilities dropped and no-new-privileges must be present. Review managed mounts rather than claiming there will always be exactly one mount.

Through sandbox exec, test for /var/run/docker.sock and the actual /run/user/DEDICATED_UID/docker.sock path; both must be unavailable. Also inspect actual mount metadata: absence of a familiar socket filename alone does not exclude an alternate mounted path. Do not probe a socket with administrative Docker API requests.

### 16.5 Execution escape and external tools

In an actual agent turn request a harmless `id` using explicit host/gateway execution, then elevated execution. Expected outcome: policy denial, not successful host execution with an innocuous result. Confirm there is no unintended node execution path. If a test unexpectedly succeeds, stop agent use and record the finding; do not continue to destructive commands.

Verify unavailable external messaging, scheduling and administrative tools against the effective tool list. A model declining verbally is not sufficient evidence of an enforced denial. Pair runtime policy inspection with an actual harmless rejected request where supported.

### 16.6 Credential isolation

Inspect container environment VARIABLE NAMES only through a redacting local parser; do not output full docker inspect/config/environment. Confirm no provider-key or Gateway-token variable is forwarded. Review mount sources and build inputs. For leakage-detection workflow tests, use a synthetic sentinel in a designated host-only test file, never a real credential. Check only the specifically generated test logs/artifacts with a tool that reports match counts rather than content.

**GATE V1:** prepare and review the redacting parser and version-appropriate live-tool test method before executing this subsection. They are deliberately not replaced with an unsafe blanket env/log dump.

**Recovery for all tests:** keep the Gateway stopped or normal agent use disabled after a failed containment test, preserve sanitized evidence and fix one control at a time. Remove only the exact synthetic files/directories recorded by the test after human review; no recursive wildcard cleanup. Never declare acceptance based on “0 critical” audit count alone.

## 17. Functional Verification

**Objective:** prove useful functionality without weakening the accepted controls. **Actors:** human operator, openclaw diagnostics and sandbox agent.

1. Confirm Dashboard authentication through the Mac SSH tunnel.
2. Send a minimal synthetic request to each configured provider, with explicit awareness of API charges.
3. Ask the main agent to read and modify a synthetic workspace document; confirm persistence and correct ownership.
4. Close the administrative/login shells. Confirm the Dashboard and service remain usable via a newly established tunnel.
5. After a separately approved reboot, reconnect and verify user-manager lingering, rootless Docker, Gateway and workspace persistence.
6. Open a fresh session and check the benign identity/preference templates. Confirm one agent and no external channels or scheduled jobs.
7. Review resource use while running one ordinary task; do not infer adequate capacity from idle memory alone.

**Expected:** all functions work without any host execution granted to the agent. **Verification:** timestamped pass/fail record without secrets. **Recovery:** use section 18; never disable sandbox, authentication or approvals to obtain a green test.

## 18. Troubleshooting

**Objective:** diagnose failures without expanding authority. **Actor:** ubuntu or openclaw according to the failing layer, not the sandbox agent administering its host.

| Symptom                         | Safe investigation                                                               | Do not do                                                      |
| ------------------------------- | -------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| Docker socket unreachable       | Check user daemon, DOCKER_HOST, user runtime dir and Gateway service environment | chmod socket globally; add docker group; switch rootful socket |
| Workspace permission denied     | Check rootless UID mapping and actual container user, bind ownership             | chmod 777; disable sandbox                                     |
| Image missing                   | Verify selected daemon and approved version-specific image build inputs          | Substitute bare Debian or an unreviewed public image           |
| systemctl --user cannot connect | Check lingering, user manager, runtime dir and session bus setup                 | Run the Gateway as root to bypass it                           |
| AppArmor/user namespace denial  | Inspect sanitized relevant events and package-supplied RootlessKit profile       | Disable AppArmor system-wide                                   |
| Dashboard unavailable           | Check SSH tunnel, authentication and loopback listener                           | Publish 18789 to the Internet                                  |
| Provider auth fails             | Check host-side supported secret configuration and provider status               | Print secrets or copy them into sandbox env                    |
| Tool denied                     | Inspect intended allowlist and session policy                                    | Enable full tools/elevated to make the error disappear         |
| Source/docs disagree            | Pin revision and examine exact version schema/source                             | Quietly use floating current examples                          |

Previously observed diagnostic commands:

```bash
# openclaw; read-only diagnostics, sanitized review of output
openclaw status
openclaw sandbox explain --agent ACTUAL_AGENT_ID
openclaw exec-policy show
openclaw security audit --deep
```

Only `openclaw doctor --lint` is appropriate when a read-only doctor check is intended. Ordinary doctor or repair modes can mutate/migrate state; do not equate them with inspection. Review logs locally with redaction rather than attaching full logs.

**Expected:** identify the layer causing failure. **Recovery:** return to last approved configuration/artifact; leave unsafe features off. Record unresolved compatibility instead of making a broad bypass.

## 19. Backup and Recovery

**Objective:** preserve identity, workspace and application state with consistent protected backups. **Actors:** ubuntu for protected storage and recovery access; openclaw for application-aware lifecycle steps.

Back up workspace, reviewed configuration, required application state and protected credentials as separate sensitivity classes. Record package/image manifests and user/service setup metadata. Do not put state archives or credentials inside the workspace, source repository or image build context.

**GATE B1:** verify the exact version's persistence format and supported consistent-backup method. Do not assume copying a live SQLite file alone captures WAL state or a coherent application snapshot. A conservative approved window stops the Gateway cleanly, verifies it has exited, backs up its state consistently with ownership/modes preserved, then restarts and checks health. Exact unit commands depend on the service name verified in G1.

Encrypt backups, restrict access, define retention and test restoration on a separate controlled target. Protect encryption keys separately. An EBS snapshot is not automatically application-consistent, and a snapshot's existence is not a restore test.

**Expected / verification:** a recoverable backup with integrity checks, recorded versions and no publicly accessible secret material. **Recovery:** restore to a dedicated account on an isolated target, restore ownership/permissions, verify rootless socket configuration and disable unintended outbound integrations before bringing it online. Never restore account-wide privileged memberships from a generic old-host image blindly.

## 20. Rollback Procedures

**Objective:** revert a failed installation or upgrade without losing administrative access or silently removing isolation. **Actor:** human administrator.

- Before provisioning: abandon the design or adjust approved infrastructure; no current laboratory rollback is needed because it was not changed.
- Before first successful service start: keep the new Gateway stopped; preserve logs and workspace; correct the failing gate. Do not destroy the instance until needed evidence/data are secured.
- After configuration change: restore the specific previously approved config/drop-in, restart the identified service in an approved window and repeat safety tests.
- After application upgrade: restore both a compatible pinned package and the pre-upgrade consistent state backup if schema migration occurred. Package-only downgrade may be unsafe.
- After sandbox-image change: restore the approved image and recreate only the affected sandbox after preserving necessary workspace data; transient container files are not guaranteed recoverable.
- If rootless integration fails: STOP deployment. Rootful Docker, docker-group access and disabling sandbox are not rollback alternatives.
- Before cloud decommissioning: obtain explicit authorization, preserve required backups and confirm no data or dependent resources are needed. No destructive deletion commands are included here.

**Verification:** previous accepted state, authenticated tunnel access and all security acceptance checks. **Risk:** changing UID or subordinate-ID mappings can invalidate stored ownership. **Recovery:** use documented identity/mapping manifests; do not recursively chown all application and container state without review.

## 21. Operational Maintenance

**Objective:** keep a known-safe deployment without background changes that were not authorized. **Actors:** human administrator for host/runtime updates; dedicated account for approved application maintenance.

- Review OS, Node, Docker and OpenClaw security updates; choose explicit versions and an approved maintenance window.
- Back up first; revalidate image source, schema, rootless mapping and tool policy after upgrades.
- Inspect resource growth, disk usage, image layers, logs and API billing. Do not prune all containers/images or delete logs automatically.
- Recheck paired administrative devices, SSH sources, unused credentials and the absence of channels/automations.
- Rotate credentials through supported secret-management flows and test with synthetic requests.
- Review template changes as code; do not interpret agent-edited instructions as permission to expand technical privileges.
- Keep source provenance and acceptance evidence with the operational record, without secrets.

No recurring automation, cron job, timer or automatic update schedule is created by this runbook. If requested later, design and authorize that schedule separately.

**Verification:** periodic checks performed on request, with dated results rather than permanent security claims. **Recovery:** section 20 and the last known-good artifact/state set. A non-sudo user and Docker are defense layers, not standalone guarantees against kernel bugs, Gateway compromise or malicious approved tools.

## 22. Acceptance Criteria

**Objective:** define a clear release decision. **Actor:** human technical reviewer. No criterion is marked passed merely because this document describes it.

| ID  | Criterion                                | Required evidence                                             | Current status |
| --- | ---------------------------------------- | ------------------------------------------------------------- | -------------- |
| A01 | Version/provenance pinned                | AMI, Node, npm integrity, source revision and image manifest  | PENDING        |
| A02 | Correct host size/OS                     | Ubuntu 24.04, t3.medium, resource review                      | PENDING        |
| A03 | Dedicated non-admin identity             | Process UID, group and sudo-policy inspection                 | PENDING        |
| A04 | Persistent services                      | Verified logout/reboot recovery of user Docker and Gateway    | PENDING        |
| A05 | Gateway private/authenticated            | Loopback listener + AWS rules + external check + tunnel login | PENDING        |
| A06 | Rootless Docker only                     | Daemon security metadata, private socket, no privileged group | PENDING        |
| A07 | Main agent sandboxed                     | Effective all mode and actual normal-tool container execution | PENDING        |
| A08 | Workspace usable/persistent              | Synthetic read/write, ownership and replacement test          | PENDING        |
| A09 | Host unmounted data inaccessible         | Synthetic canary denied via file tools and exec               | PENDING        |
| A10 | No socket access from sandbox            | Mount review and socket-path negative tests                   | PENDING        |
| A11 | No host/elevated execution               | Enforced harmless negative tests, not verbal refusal          | PENDING        |
| A12 | Runtime restrictions enforced            | None network, read-only root, cap drop, no-new-privileges     | PENDING        |
| A13 | Secrets excluded                         | Redacted env/mount/build-input review and synthetic checks    | PENDING        |
| A14 | Both APIs functional                     | Approved minimal synthetic requests                           | PENDING        |
| A15 | Single agent and reviewed identity       | Agent count, identity sync, fresh-session preference test     | PENDING        |
| A16 | No extra channels/integrations/schedules | Effective configuration and inventory review                  | PENDING        |
| A17 | Backup and rollback demonstrated         | Consistent backup and separate restore test                   | PENDING        |
| A18 | Version-sensitive gaps resolved          | P1-P4, D1-D4, G1, W1-W2, I1, T1-T3, A1, V1, B1                | PENDING        |

**Release decision:** NOT APPROVED FOR EXECUTION yet. This is a complete review draft, not a fully validated deployment recipe. Missing installation/provider/policy syntax is intentionally visible rather than guessed. Record the approver, date, exact versions and residual risks before execution.

Residual risks include shared kernel/container isolation limits, Gateway authority over its own credentials and rootless daemon, writable instruction files, provider data disclosure/cost, host-side tool egress, mutable upstream artifacts and limited initial instance capacity.

## 23. References and Sources

**Objective:** make provenance and uncertainty auditable. **Actor:** reviewer. No additional retrieval was performed when finishing this document after the owner's “no more searches” instruction.

### Evidence actually available from the prior investigation

- **OC-LOCAL:** installed /home/ubuntu/.npm-global/lib/node_modules/openclaw/package.json: version 2026.11.2, Node engines >=24.16.0 <25 || >=26.1.0; no gitHead recorded in the inspected output. Local compiled source includes dist/docker-CtRxN_p0.mjs (container create arguments and bookworm-slim missing-image branch) and dist/podman-runtime-qXIoqlCL.mjs (Docker CLI invocation). Bundle filenames are installation-specific, not stable APIs.
- **OC-USER-SOURCE:** retrieved source https://raw.githubusercontent.com/openclaw/openclaw/b556fdf457b026574bf0f1e7b4914b15f3f179ce/src/agents/sandbox/docker-user.ts — explicit docker.user overrides host workspace UID/GID auto-selection. Release-to-package provenance still requires P1.
- **OC-DOCKER:** https://docs.openclaw.ai/gateway/sandboxing/docker-backend — prior retrieval covers rootless DOCKER_HOST selection, restricted defaults, container user behavior and managed mounts. Current documentation is not an immutable 2026.11.2 reference.
- **D-MAPPING:** https://docs.docker.com/engine/security/rootless/uid-gid-mapping.md — prior retrieval explains UID 0 to host daemon-user mapping and subordinate-ID behavior.
- **OC-SANDBOX:** local docs/gateway/sandboxing.md under the installed package; official counterpart https://docs.openclaw.ai/gateway/sandboxing — host Gateway versus sandbox tools, mode/scope and escape routes.
- **OC-POLICY:** local docs/gateway/config-tools/tool-policy.md, especially tools.elevated; local docs/tools/exec-approvals.md and docs/cli/approvals.md. Official entry points https://docs.openclaw.ai/tools/exec-approvals and https://docs.openclaw.ai/gateway/sandbox-vs-tool-policy-vs-elevated.
- **OC-AGENTS:** local docs/cli/agents.md; https://docs.openclaw.ai/cli/agents — identity files and set-identity semantics.
- **OC-BOOTSTRAP:** local docs/start/bootstrapping.md; https://docs.openclaw.ai/start/bootstrapping — initial files and configured-workspace cleanup behavior.
- Existing laboratory record: /home/ubuntu/.openclaw/workspace/docs/laboratorio-acceso-seguridad.md. It describes the OLD host and is not evidence that the new installation passes these tests.

### Identified official references requiring final version-specific review

These are review targets; listing them does not claim that every page or command was inspected before this draft was completed.

- **OC-WORKSPACE:** https://docs.openclaw.ai/gateway/sandboxing/workspace-access
- **OC-IMAGES:** https://docs.openclaw.ai/gateway/sandboxing/images-and-setup
- Release/source: https://github.com/openclaw/openclaw/releases/tag/v2026.11.2 and https://github.com/openclaw/openclaw
- **D-UBUNTU:** https://docs.docker.com/engine/install/ubuntu/
- **D-ROOTLESS:** https://docs.docker.com/engine/security/rootless/
- **D-TROUBLE:** https://docs.docker.com/engine/security/rootless/troubleshoot/
- Rootless operating tips: https://docs.docker.com/engine/security/rootless/tips/
- AWS EC2 documentation: https://docs.aws.amazon.com/ec2/
- Node release verification: https://nodejs.org/en/download
- systemd user persistence: loginctl(1) and systemd.exec(5) manuals for the installed Ubuntu release.
- OpenAI API documentation: https://platform.openai.com/docs/
- Google Gemini API documentation: https://ai.google.dev/gemini-api/docs

**Review rule:** prefer exact-version source/schema over floating examples; resolve contradictions explicitly. Do not transform a reference link into an assertion that an untested command is verified. Once gates are resolved, record execution evidence separately from documentary verification.
