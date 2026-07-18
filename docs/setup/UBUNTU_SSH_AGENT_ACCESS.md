# Ubuntu WSL2 SSH Access for Autonomous Agents

This guide configures Ubuntu 24.04 under WSL2 so trusted coding agents can work inside the CharOS development environment through SSH.

| Item | Value |
| --- | --- |
| Host | Windows 11 |
| Linux environment | WSL2 / Ubuntu 24.04 |
| User | `abi` |
| Repository | `~/Projects/CharOS` |
| SSH server | OpenSSH |

The intended development stack includes Git, GitHub CLI, Docker Desktop's Windows backend, Node.js, Python, Windows-hosted Ollama, and future Cognee and Obsidian integrations.

## Table of contents

1. [Why SSH](#why-ssh)
2. [Architecture](#architecture)
3. [Install and start OpenSSH](#install-and-start-openssh)
4. [Configure Windows key access](#configure-windows-key-access)
5. [Verify access](#verify-access)
6. [Security hardening](#security-hardening)
7. [WSL networking limitations](#wsl-networking-limitations)
8. [Agent integration](#agent-integration)
9. [Recommended workflow](#recommended-workflow)
10. [Automation notes](#automation-notes)
11. [Troubleshooting](#troubleshooting)
12. [Verification checklist](#verification-checklist)
13. [Future improvements](#future-improvements)

## Why SSH

SSH provides a secure, reproducible command boundary into Ubuntu. It prevents each agent from needing its own WSL/Windows API integration and gives local or remote, self-hosted agents the same controlled interface.

Benefits:

- **Secure:** public-key authentication and server policy control who can connect.
- **Reproducible:** agents use the same Linux paths, permissions, shells, and tools.
- **Tool-agnostic:** Codex CLI, Claude Code, Gemini CLI, OpenHands, and future CharOS agents can use a standard SSH client.
- **Local-first:** Windows-hosted agents can connect over localhost without a cloud service.
- **Extensible:** the endpoint can be made available to a trusted remote machine through a VPN.

SSH does not give a browser-based chat direct access to a private computer. A local companion process, CLI, or explicitly connected tool must hold an authorized private key and initiate the session.

## Architecture

```text
Developer / trusted local agent
              |
              | SSH (key-authenticated)
              v
Windows 11 localhost or private VPN/LAN endpoint
              |
              v
WSL2: Ubuntu 24.04 (user: abi)
              |
              v
~/Projects/CharOS
              |
              +-- Git / GitHub CLI
              +-- Docker Desktop integration
              +-- Node.js and Python
              +-- Ollama on the Windows host
              +-- Cognee and Obsidian integrations (planned)
```

```text
Planner
   |
   v
SSH tool / client
   |
   v
Ubuntu command boundary
   |
   +--> Git --> GitHub
   +--> Docker --> Docker Desktop backend
   +--> Python / Node --> tests and build tools
   +--> Ollama --> Windows-hosted local inference
   +--> Cognee --> local knowledge graph (planned)
   |
   v
CharOS repository and result report
```

The SSH server starts sessions as `abi`; agents inherit only that account's permissions. Do not provide unrestricted sudo access simply because a caller is an AI agent.

## Install and start OpenSSH

Run these commands in the Ubuntu shell as `abi`:

```bash
sudo apt update
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
```

Verify the listener and discover the current WSL address:

```bash
sudo ss -tlnp | grep ssh
hostname -I
```

Ubuntu 24.04 WSL installations normally support systemd when enabled. If `systemctl` reports that systemd is not running, set the following in `/etc/wsl.conf`:

```ini
[boot]
systemd=true
```

Then restart WSL from Windows PowerShell and relaunch Ubuntu:

```powershell
wsl --shutdown
```

Do not use an unmanaged background `sshd` process as the permanent replacement for service startup.

## Configure Windows key access

Generate a dedicated Windows key in PowerShell. Use a passphrase:

```powershell
ssh-keygen -t ed25519 -f "$env:USERPROFILE\.ssh\charos_wsl_ed25519" -C "charos-wsl-agent"
type $env:USERPROFILE\.ssh\charos_wsl_ed25519.pub
```

Copy the public-key line into Ubuntu's `~/.ssh/authorized_keys`:

```bash
install -d -m 700 ~/.ssh
nano ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

Never copy the private key into WSL or the repository. Validate the daemon configuration, then restart it:

```bash
sudo sshd -t
sudo systemctl restart ssh
```

## Verify access

From Windows PowerShell, substitute the address returned by `hostname -I`:

```powershell
ssh -i "$env:USERPROFILE\.ssh\charos_wsl_ed25519" abi@<WSL_IP>
```

Expected result:

```text
abi@ABI-LEGION:~$
```

Verify the project toolchain in the SSH session:

```bash
cd ~/Projects/CharOS
git status --short --branch
git remote -v
gh auth status
docker version
node --version
python3 --version
```

Also test a non-interactive command, which is how many agents execute tasks:

```powershell
ssh -i "$env:USERPROFILE\.ssh\charos_wsl_ed25519" abi@<WSL_IP> "cd ~/Projects/CharOS && git status --short --branch"
```

## Security hardening

Complete the key-authentication test above before changing authentication policy. Create a reviewable SSH configuration fragment:

```bash
sudo tee /etc/ssh/sshd_config.d/90-charos-agent-access.conf >/dev/null <<'EOF'
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
AllowUsers abi
X11Forwarding no
AllowTcpForwarding no
MaxAuthTries 3
LoginGraceTime 30
EOF
sudo sshd -t
sudo systemctl restart ssh
```

This disables password and root login, limits access to `abi`, and blocks forwarding. Re-enable forwarding only for a documented, approved tunnel.

Additional recommendations:

- Create separate, named keys for people, local automation, and each long-lived agent; revoke one key by deleting its line from `authorized_keys`.
- Protect private keys with passphrases and the Windows OpenSSH agent. Never commit keys, tokens, `.env` files, or Obsidian vault secrets.
- Use a dedicated least-privilege Linux user when agent tasks go beyond supervised coding.
- Require approval for destructive operations, credential access, publishing, or egress beyond approved endpoints.
- Keep Windows, WSL, and Ubuntu packages patched; review SSH logs and config changes periodically.
- Allow localhost, private LAN, or VPN only. Never configure public router port-forwarding for SSH.

If UFW is enabled, permit only the intended private source:

```bash
sudo ufw allow from <TRUSTED_PRIVATE_SUBNET> to any port 22 proto tcp
sudo ufw status verbose
```

Windows Defender Firewall governs inbound traffic before it reaches WSL; keep it closed unless private-LAN or VPN access is explicitly required.

## WSL networking limitations

In WSL's default NAT networking, its virtual IP may change after WSL restarts. Treat `hostname -I` as discovery, not as a stable endpoint.

| Option | Advantages | Limitations |
| --- | --- | --- |
| Windows localhost forwarding | Simple for Windows-local agents; no LAN exposure | Depends on WSL/Windows configuration |
| Windows `netsh interface portproxy` | Stable Windows port such as `localhost:2222` | Refresh required after WSL IP changes; expands exposure if not loopback-only |
| Mirrored networking | Better host/LAN integration | Requires compatible WSL/Windows version and firewall review |
| Tailscale or WireGuard | Stable, authenticated remote access without public port-forwarding | Adds network and key lifecycle management |

For host-local automation, prefer a loopback-only endpoint such as `127.0.0.1:2222`. If using a portproxy, bind it only to loopback unless a private-network requirement is approved, and automate rule refresh after the WSL IP changes.

Example Windows SSH client profile in `%USERPROFILE%\.ssh\config`:

```sshconfig
Host charos-wsl
    HostName 127.0.0.1
    Port 2222
    User abi
    IdentityFile ~/.ssh/charos_wsl_ed25519
    IdentitiesOnly yes
```

After configuration, agents use `ssh charos-wsl` instead of embedding an IP in prompts or scripts.

## Agent integration

Local agents use SSH as an execution transport: connect, change to the repository, run a scoped command, collect output, and report results. They should not receive unrestricted sudo credentials or an unrestricted shell merely because they are autonomous.

```text
Task request
   |
   v
Planner creates bounded plan
   |
   v
SSH tool connects as abi (key only)
   |
   v
~/Projects/CharOS
   |
   +--> inspect --> test --> patch --> verify
   |                         |
   |                         +--> commit/push only when authorized
   v
Return logs, changed files, and verification result
```

Codex CLI, Claude Code, Gemini CLI, OpenHands, and a future CharOS runtime can follow this pattern when they run on a machine that reaches the SSH endpoint and holds an authorized private key. ChatGPT in a browser cannot directly SSH to a private local computer.

For CharOS, model this as an `SshTool` with explicit host allowlists, pinned host keys, repository-path allowlists, permitted command classes, timeouts, approval gates, and audit events. Keep keys and GitHub tokens outside model context.

## Recommended workflow

```text
AI receives a scoped task
        |
        v
SSH into Ubuntu as abi
        |
        v
cd ~/Projects/CharOS
        |
        v
inspect state and fetch approved updates
        |
        v
install declared dependencies; run tests
        |
        v
make focused changes; verify them
        |
        v
commit and push only with explicit authorization
        |
        v
report commands, changed files, tests, and blockers
```

Start each task with read-only preflight:

```bash
cd ~/Projects/CharOS
git status --short --branch
git remote -v
gh auth status
```

Before changing dependencies or Docker resources, inspect first. Never run broad cleanup such as `docker system prune -a`, `git reset --hard`, or recursive deletion without a precisely approved scope.

## Automation notes

- Use SSH aliases and pinned `known_hosts` entries. Do not automatically accept changed host keys in unattended jobs.
- Use batch mode in non-interactive jobs: `ssh -o BatchMode=yes charos-wsl '<command>'`.
- Discover the WSL IP only in local setup automation; do not store it in source code or prompts.
- Confirm systemd service startup before depending on it after WSL reboots.
- Use Docker Desktop's documented WSL integration; do not start a second Docker daemon in Ubuntu without an architectural need.
- Test Ubuntu-to-Windows Ollama access deliberately. Never expose Ollama beyond trusted local networking without an authentication and network-control design.
- Make Git and GitHub checks read-only preflight. Require a policy gate for commits, pushes, repository changes, and external `gh` actions.

## Troubleshooting

### Permission denied (publickey)

Confirm the intended key is selected:

```powershell
ssh -vvv -i "$env:USERPROFILE\.ssh\charos_wsl_ed25519" abi@<WSL_IP>
```

In Ubuntu, correct directory modes and ownership:

```bash
ls -ld ~/.ssh
ls -l ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
sudo chown -R abi:abi ~/.ssh
```

### A password is still requested

The key was not accepted. Check the identity path, exact public-key line, file modes, and `sudo sshd -t`. Do not disable passwords until a separate key-authenticated session is working.

### Host key verification failed

If the instance was recreated or host keys changed, remove only the stale target entry:

```powershell
ssh-keygen -R <WSL_IP>
```

Verify the new fingerprint out of band before accepting it. Never disable host-key checking for automation.

### SSH is inactive or systemd is disabled

```bash
sudo systemctl status ssh
sudo journalctl -u ssh --no-pager -n 100
cat /etc/wsl.conf
```

Ensure `/etc/wsl.conf` contains `[boot]` and `systemd=true`, run `wsl --shutdown` from Windows, then relaunch Ubuntu.

### Port already in use

```bash
sudo ss -tlnp | grep ':22'
sudo systemctl status ssh
```

Identify the owning process before changing ports. If another approved service owns port 22, configure a documented alternative port and update the SSH client profile.

### Connection timed out or the WSL IP changed

Re-run `hostname -I`, check the listener, then test the exact current IP from Windows:

```bash
hostname -I
sudo ss -tlnp | grep ssh
```

For durable access, use localhost forwarding, a maintained portproxy refresh script, mirrored networking, or a VPN hostname; do not hard-code an old WSL IP.

### Windows cannot access the private key

Keep it in `%USERPROFILE%\.ssh` with access restricted to the Windows user. If an imported key has incorrect ACLs, repair them or generate a new dedicated key; never relocate it into the repository or a shared directory.

### Docker, GitHub CLI, or Ollama is unavailable after SSH login

SSH's non-interactive environment can differ from an interactive terminal:

```bash
command -v git gh docker node python3 ollama
printf '%s\n' "$PATH"
```

Enable Docker Desktop's WSL integration for Ubuntu 24.04. Authenticate GitHub CLI interactively as `abi` with `gh auth status`; keep its credentials outside the repository.

## Verification checklist

- [ ] `openssh-server` is installed in Ubuntu 24.04.
- [ ] `sudo systemctl status ssh` shows an active service.
- [ ] `sudo ss -tlnp | grep ssh` shows the intended listener.
- [ ] A dedicated Windows Ed25519 key exists; its private half is not in WSL or the repository.
- [ ] Its public half is in `/home/abi/.ssh/authorized_keys`.
- [ ] `~/.ssh` is mode `700`, `authorized_keys` is mode `600`, and both belong to `abi`.
- [ ] A new PowerShell session connects successfully over SSH.
- [ ] A non-interactive SSH command can read `~/Projects/CharOS`.
- [ ] Key authentication works before password auth is disabled.
- [ ] `sshd -t` passes after hardening; root login is disabled.
- [ ] The endpoint is localhost, private LAN, or VPN only—not the public internet.
- [ ] Git state, GitHub CLI auth, Docker integration, Node, and Python are verified in the SSH session.
- [ ] Agent permissions and commit/push approval policy are documented.

## Future improvements

- Stable `localhost:2222` endpoint with automated WSL IP refresh.
- Reverse SSH through a trusted authenticated relay when direct private networking is unavailable.
- Tailscale or WireGuard for remote access without public SSH exposure.
- WSL mirrored networking after firewall and version compatibility review.
- Approved remote-development profiles and GitHub Actions CI integration.
- A CharOS `SshTool` with host allowlists, host-key pinning, scoped commands, timeouts, and audit logs.
- Multi-agent execution in isolated worktrees or containers instead of shared mutable directories.
- OS-backed secret management for SSH, GitHub, Ollama, Cognee, and Obsidian credentials, with rotation and revocation.

