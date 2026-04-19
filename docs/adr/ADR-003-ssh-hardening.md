# ADR-003: SSH Hardening and Key-Based Authentication Enforcement for SentinelAI Production Server

## Status
Accepted

## Date
2026-04-19

## Context
* Prior to this decision,openssh-client is installed but openssh-server is not, also every other control  built so far operates inside the server (user accounts, sudo policy, hostname — these matter once someone is already on the machine), SSH is different. SSH is the network-facing attack surface. It is the service that the entire internet can reach and attempt to authenticate against.
* A new server appearing on the internet (like our production server) with SSH on port 22, automated scanners are attempting to log in. They try root with common passwords. They try admin. They try credential lists from previous data breaches. This is not theoretical. This happens to every server, every time, without exception.
* The hardened SSH configuration will make automated attacks useless and targeted attacks dramatically harder.

## Decision
* Installing openssh-server, that is the package that runs the daemon and lets people SSH in
* Generating an SSH key pair for user chiheb using ssh-keygen with the ed25519 algorithm (Ed25519 is a modern elliptic curve algorithm that is faster, shorter, and considered more secure than RSA 4096) and a strong passphrase to encrypt private key file and a comment as the user email for tracability
* Install the public key by copying it into chiheb authorized_keys file with the correct permissions for .ssh/ directory (700owner only) and for authorized_keys (600owner read/write only) otherwise SSH will silently refuse to use keys with wrong permissions as a security measure.
* Create /etc/ssh/sshd_config.d/99-hardening.conf with all the necessary directives:Disable Root Login (root is the most accout targeted , every attacker tries root first because its name is always the same on every Linux system), Disable Password Authentication (Passwords can be guessed, brute-forced, phished, and leaked in data breaches. SSH keys cannot This single change eliminates the entire class of brute-force and credential-stuffing attacks), Enforce SSH Protocol and Limit Authentication Methods, Change the Default Port to Port 2222 ( Changing the SSH port from 22 to 2222 is a move to reduce attack surface visibility it effectively eliminates the noise of automated bots. This clears your logs of thousands of failed login attempts, making it much easier to identify and respond to genuine, targeted threats), Set Idle Timeout ( to avoid physical access), Restrict Login to Specific Users (ssh only with chiheb),Disable Unnecessary Features (like X11Forwarding , AgentForwarding ...), Harden Cryptographic Algorithms (explicitly whitelist only strong modern algorithms and reject everything else) . The 99- prefix means it loads last and overrides any earlier configuration.
* Disable Socket Activation, Use Traditional Service because Ubuntu 24.04 uses systemd socket activation by default. Instead of ssh.service listening permanently on a port, ssh.socket wich owns port 22 and is triggering ssh.service. When socket activation is in play, the socket hands an already-open file descriptor to sshd — meaning sshd never reads the Port directive from your config because the socket already decided the port. Your Port 2222 in 99-hardening.conf is being ignored entirely
* create /run/sshd directory manually so sshd can satisfy its security check (OpenSSH expects a specific directory to exist for security reasons )

## Consequences
* Authentication is strictly bound to the possession of the private key and knowledge of its passphrase. Losing the private key results in total lockout, as password-based recovery is disabled at the network level.
* All administrative connections must now specify a non-standard port (-p 2222). This must be documented in all internal team onboarding guides to prevent "Connection Refused" support tickets.
* By disabling Systemd Socket Activation, we have moved away from the Ubuntu 24.04 default. This increases the configuration surface area (requiring ssh.service management), but provides a predictable environment where the sshd_config is the "single source of truth."
* Idle sessions are now automatically terminated after 10 minutes of inactivity. While this may cause minor inconvenience during long debugging sessions, it significantly reduces the risk of hijacked "ghost" sessions.

## Alternatives Considered
* Maintaining Port 22 to simplify connection strings. Rejected because the volume of automated "noise" in the logs makes it nearly impossible to identify legitimate, targeted brute-force attempts. Port 2222 provides cleaner logs for security auditing.
* Using RSA 4096-bit Keys as it is the most widely compatible algorithm. Rejected in favor of Ed25519 because Ed25519 offers a higher security margin with significantly smaller key sizes and faster performance.
* Maintaining Socket Activation so that the ssh.socket unit listens on port 2222. Rejected because socket activation adds an extra layer of Systemd abstraction that makes troubleshooting SSH issues more complex in a production environment compared to a standard, always-on service.
