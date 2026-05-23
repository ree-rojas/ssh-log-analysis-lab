---

# Fail2Ban Integration and Automated SSH Protection

After validating manual SSH log analysis and brute force detection, the environment was extended using Fail2Ban to automate defensive responses against suspicious authentication behavior.

Fail2Ban continuously monitors authentication logs and dynamically creates temporary firewall rules when malicious patterns are detected.

The objective of this stage was to:
- monitor SSH authentication failures,
- automatically detect brute force behavior,
- temporarily ban suspicious IP addresses,
- and validate automated threat mitigation.

---

# Fail2Ban Installation

Package installation:

```bash
sudo apt update && sudo apt install fail2ban -y
```

Service reload:

```bash
sudo systemctl reload fail2ban
```

---

# Fail2Ban Configuration Structure

Main configuration directory:

```bash
/etc/fail2ban/
```

Important files:

| File | Description |
|---|---|
| jail.conf | Default Fail2Ban configuration |
| jail.local | Local custom configuration |
| fail2ban.log | Fail2Ban activity logs |

The `jail.conf` file belongs to the package itself and should not be modified directly.

Instead, custom configurations are created inside:

```bash
/etc/fail2ban/jail.local
```

**This preserves the original package configuration and prevents issues during future updates.**

---

# SSH Jail Configuration

The SSH jail was configured manually inside:

```bash
sudo nano /etc/fail2ban/jail.local
```

Configuration used:

```ini
[sshd]
enabled = true
port = 22
logpath = /var/log/auth.log
maxretry = 3
findtime = 60
bantime = 600
```

Configuration explanation:

| Parameter | Description |
|---|---|
| enabled | Enables the SSH jail |
| port | SSH internal service port |
| logpath | Authentication log monitored by Fail2Ban |
| maxretry | Maximum failed attempts allowed |
| findtime | Time window for counting failures |
| bantime | Temporary ban duration in seconds |

With this configuration:
- if an IP generates 3 failed SSH authentications,
- within 60 seconds,
- the IP is automatically blocked for 10 minutes.

---

# Jail Validation

After reloading the service, active jails were verified using:

```bash
sudo fail2ban-client status
```

Example output:

```txt
Status
|- Number of jail: 1
`- Jail list: sshd
```

This confirms that:
- one jail (module) is active,
- and the SSH monitoring module was loaded successfully.

---

# SSH Jail Inspection

Detailed SSH jail information was inspected using:

```bash
sudo fail2ban-client status sshd
```

This command displays:
- failed authentication counters,
- active bans,
- historical bans,
- and currently blocked IP addresses.

![Fail2Ban SSH Jail Status](images/fail2ban-status.png)

---

# Output Interpretation

| Field | Meaning |
|---|---|
| Currently failed | Current failed attempts detected |
| Total failed | Total authentication failures detected |
| Currently banned | Number of IPs currently banned |
| Total banned | Historical number of bans performed |
| Banned IP list | IPs actively blocked at that moment |

Important observation:

The `Total banned` **counter remains recorded even after the ban expires.**

However, `Banned IP list` only displays IP addresses **currently under active ban.**

After the configured `bantime` expires:
- the IP is automatically unbanned,
- removed from the active list,
- and firewall access is restored.

---

# Automated Brute Force Detection

A brute force simulation was executed again to validate automated protection behavior.

Attack simulation:

```bash
for i in {1..10}; do sshpass -p "123" ssh -o StrictHostKeyChecking=no -p 2222 fakeuser@localhost; done
```

During the attack:
- multiple authentication failures were generated,
- Fail2Ban detected the malicious pattern,
- and the firewall automatically blocked the attacking IP.

After the threshold was exceeded, SSH connections started returning:

```txt
kex_exchange_identification: read: Connection reset by peer
```

This occurred because the attacking IP address was already under active firewall restriction.

![Automated SSH Blocking](images/fail2ban-block.png)

---

# Fail2Ban Log Monitoring

Fail2Ban events can be monitored in real time using:

```bash
sudo tail -f /var/log/fail2ban.log
```

The log records:
- detected attacks,
- active bans,
- unban events,
- and jail activity.

![Fail2Ban Log Monitoring](images/fail2ban-log.png)

---

# Manual IP Unban

A banned IP address can also be manually removed from the jail using:

```bash
sudo fail2ban-client set sshd unbanip 10.0.2.2
```

This immediately:
- removes the firewall restriction,
- deletes the IP from the jail,
- and restores SSH connectivity.

---

# Concepts Practiced

- Automated threat mitigation
- Fail2Ban integration
- SSH protection
- Dynamic firewall actions
- Brute force prevention
- Authentication monitoring
- Temporary IP banning
- Real-time defensive response

---
