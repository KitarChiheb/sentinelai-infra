# Runbook: SSH Hardening and Key-Based Authentication
## Purpose
* This runbook describes the procedure for installing and hardening the OpenSSH server on Ubuntu 24.04. It enforces key-based authentication, disables root login, and shifts the service to port 2222 to reduce attack surface visibility.

## Prerequisites
* You must have sudo access.
* You must have an open terminal session maintained throughout the process as a "break-glass" fallback.
* You must have generated an Ed25519 key pair for the chiheb account.
* **Backup:** sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

## Risk Assessment
* Disabling PasswordAuthentication before verifying the SSH key works will result in a permanent lockout.
* Ubuntu 24.04's default 'Socket Activation' will override port settings unless manually disabled.
* Incorrect syntax in the hardening file will prevent the SSH service from starting.

## Procedure
### Step 1 — Install openssh-server
* Run: sudo apt update && sudo apt install openssh-server -y
* Verify initial status: sudo systemctl status ssh
* Expected: Service is loaded but might be managed by ssh.socket.

### Step 2 — Generate and Install SSH Keys
* Switch to user: su - chiheb
* Generate key: ssh-keygen -t ed25519 -C "kitarchiheb11@gmail.com"
* Set directory permissions: chmod 700 ~/.ssh
* Install public key: cp ~/.ssh/id_ed25519.pub ~/.ssh/authorized_keys
* Set file permissions: chmod 600 ~/.ssh/authorized_keys
* Verify: ls -la ~/.ssh/ (Expected: 700 for .ssh, 600 for authorized_keys and id_ed25519).

### Step 3 — Create Hardening Configuration
* Open the hardening file: sudo nano /etc/ssh/sshd_config.d/99-hardening.conf
* Paste the following content:
PermitRootLogin no
PasswordAuthentication no
AuthenticationMethods publickey
Port 2222
ClientAliveInterval 300
ClientAliveCountMax 2
AllowUsers chiheb
X11Forwarding no
AllowAgentForwarding no
AllowTcpForwarding no
PrintMotd no
KexAlgorithms curve25519-sha256,diffie-hellman-group14-sha256
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com

### Step 4 — Disable Socket Activation (Mandatory for Port 2222)
* Run the following to ensure the config file is respected:
* sudo systemctl stop ssh.socket
* sudo systemctl disable ssh.socket
* sudo systemctl enable ssh.service

### Step 5 — Validate and Apply
* Check for syntax errors: sudo sshd -t
* Note: If missing /run/sshd, run: sudo mkdir -p /run/sshd
* Restart service: sudo systemctl restart ssh.service
* Verify port: ss -tlnp | grep 2222

### Step 6 — Verification
* From a NEW terminal: ssh -p 2222 -i ~/.ssh/id_ed25519 chiheb@localhost
* Confirm Root Rejection: ssh -p 2222 root@localhost (Expected: Permission denied).

## Rollback Procedure
### Step 1 — Revert Configuration
* sudo rm /etc/ssh/sshd_config.d/99-hardening.conf
* sudo systemctl enable ssh.socket
* sudo systemctl start ssh.socket
### Step 2 — Restart Service
* sudo systemctl restart ssh

## Notes & Known Issues
* Ubuntu 24.04 Socket Activation: The Port directive in sshd_config is ignored unless ssh.socket is disabled.
* Missing /run/sshd: This directory is required for privilege separation and must exist for sshd -t to pass.
