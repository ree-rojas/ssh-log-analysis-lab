# SSH Hardening and Secure Remote Access

After validating brute force detection and automated blocking with Fail2Ban, the SSH service itself was hardened to reduce the attack surface and improve remote access security.

The objective of this stage was to:

- reduce SSH exposure,
- limit authentication abuse,
- restrict remote administrative access,
- implement cryptographic authentication,
- and improve secure remote session management.

---

**Before starting the SSH hardening process, Fail2Ban was temporarily disabled in order to observe SSH behavior without interference from automated blocking rules.**

Since Fail2Ban actively monitors authentication failures and automatically bans IP addresses after repeated failed attempts, keeping it enabled during the hardening tests could interfere with the validation of native SSH protections such as:

- `MaxAuthTries`
- session disconnection behavior
- authentication limits
- and access restriction policies

For this reason, the SSH jail inside `jail.local` was temporarily disabled during the tests.

---

# Backup Before Changes

Before modifying the SSH daemon configuration, a backup of the original configuration file was created.

Command used:

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```

![backup-sshd-config](images/backup-sshd-config.png)

---

# SSH Hardening Configuration

The SSH daemon configuration file was opened using:

```bash
sudo nano /etc/ssh/sshd_config
```

The following security directives were modified or added.

---

## Disable Root Remote Login

```bash
PermitRootLogin no
```

This prevents direct remote login using the root account.

Even if an attacker discovers the root password, SSH access will still be denied.

---

## Limit Authentication Attempts

```bash
MaxAuthTries 3
```

This limits the number of failed authentication attempts per connection.

After three invalid attempts, the SSH server forcibly disconnects the client.

This helps reduce brute force effectiveness.

---

## Session Timeout Configuration

```bash
ClientAliveInterval 300
ClientAliveCountMax 0
```

These directives control inactive SSH sessions.

- `ClientAliveInterval 300`
  defines a 5-minute inactivity check interval.

- `ClientAliveCountMax 0`
  forces immediate disconnection if the client stops responding.

This reduces the risk of abandoned administrative sessions remaining open indefinitely.

---

## Restrict Allowed SSH Users

```bash
AllowUsers renan
```

Only explicitly authorized users are allowed to authenticate remotely through SSH.

This reduces unnecessary exposure of local system accounts.


---

# Configuration Validation

Before reloading the SSH service, the configuration syntax was validated using:

```bash
sudo sshd -t
```

**This step is critical because syntax errors in `sshd_config` may prevent remote access entirely.**

If the command produces no output, the configuration is considered valid.

---

# Reloading the SSH Service

After validation, the SSH daemon was reloaded without terminating active sessions.

Commands used:

```bash
sudo systemctl reload ssh
sudo systemctl status ssh
```

**The `reload` command was used instead of `restart` to apply the new configuration without interrupting active SSH sessions.**

![ssh-service-running](images/ssh-service-running.png)

---

# SSH Key Authentication

To improve authentication security, passwordless authentication using asymmetric cryptography was implemented.

Instead of relying exclusively on passwords, the environment now uses:

- a public key stored on the server,
- and a private key stored on the client machine.

---

## Generating the SSH Key Pair

On the client machine, the following command was executed:

```bash
ssh-keygen -t ed25519
```

The Ed25519 algorithm was selected because it is modern, lightweight, and considered more secure than older RSA defaults.

Generated files:

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

Important distinction:

- `id_ed25519`
  → private key **(must never be shared)**

- `id_ed25519.pub`
  → public key (safe to distribute)

![ssh-key-generation](images/ssh-key-generation.png)

---

## Installing the Public Key on the Server

The public key was copied to the server using:

```bash
ssh-copy-id -p 2222 renan@localhost
```

This automatically:

- creates the `.ssh` directory if necessary,
- creates the `authorized_keys` file,
- copies the public key,
- and applies proper permissions.

---

# Testing Key-Based Authentication

After key installation, SSH access was tested again.

Command used:

```bash
ssh -p 2222 renan@localhost
```

The server successfully authenticated the client using the private key instead of requesting the Linux account password.

---

# Password Authentication Hardening

To reduce brute force exposure even further, password authentication was disabled.

Configuration applied:

```bash
PasswordAuthentication no
```

This forces SSH authentication to rely exclusively on cryptographic keys.

The objective is to eliminate:

- password guessing,
- credential spraying,
- weak password abuse,
- and basic automated brute force attacks.

![password-authentication-disabled](images/password-authentication-disabled.png)

---

# Security Concepts Reinforced

This hardening stage reinforced several important cybersecurity concepts:

- attack surface reduction,
- authentication hardening,
- secure remote administration,
- session control,
- asymmetric cryptography,
- authorization policies,
- and layered defensive security.

---

# Observed Security Improvements

After applying the hardening measures:

- root SSH access was disabled,
- authentication attempts became limited,
- inactive sessions became controlled,
- remote access was restricted to authorized users,
- password authentication exposure was minimized,
- and cryptographic authentication was successfully implemented.

---

# Next Steps

Planned future improvements include:

- firewall segmentation with UFW,
- service exposure reduction,
- centralized log analysis,
- SIEM integration,
- alert correlation,
- and advanced threat monitoring.

---
