# Ubuntu Server Hardening Lab — CIS Benchmark Aligned

## Overview
A hands-on Ubuntu server hardening project built from scratch
as an AppSec student, following CIS Benchmark controls and
industry best practices to secure a Linux server against
real world attacks.

## CIS Controls Covered
| Control | Task |
|---------|------|
| **CIS Control 3** | File permission hardening |
| **CIS Control 4** | SSH and firewall hardening |
| **CIS Control 5 & 6** | User and authentication hardening |
| **CIS Control 7** | System update and patching |
| **CIS Control 8** | Logging and auditing |

## Lab Requirements
- Ubuntu Server (18.04 / 20.04 / 24.04)
- User with sudo privileges
- Internet access

---

## Task 1 — System Update & Patching (CIS Control 7)

### Update package list
```bash
sudo apt update
```

### Upgrade system
```bash
sudo apt upgrade -y
```

### Enable automatic updates
```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure unattended-upgrades
```

**Why:** Attackers exploit known vulnerabilities in outdated
packages. Automatic updates ensure your system is always
patched without relying on human memory.

---

## Task 2 — User & Authentication Hardening (CIS Control 5 & 6)

### List all users
```bash
cut -d: -f1 /etc/passwd
```

### Lock root account
```bash
sudo passwd -l root
```

### Create secure admin user
```bash
sudo adduser love
sudo usermod -aG sudo love
```

### Enforce password policy
```bash
sudo apt install libpam-pwquality -y
sudo vim /etc/security/pwquality.conf
```
Configure:
minlen = 12
dcredit = -1
ucredit = -1
lcredit = -1
ocredit = -1


**Why:** Root is the #1 target for attackers. Strong password
policies prevent dictionary and brute force attacks.

---

## Task 3 — SSH Hardening (CIS Control 4)

### Edit SSH config
```bash
sudo vim /etc/ssh/sshd_config:
```
### Apply hardening settings
PermitRootLogin no
PasswordAuthentication no
PermitEmptyPasswords no
MaxAuthTries 3

### Restart SSH
```bash
sudo systemctl restart ssh
```

**Why:** SSH is the #1 attack vector on servers. Disabling
root login and password authentication forces key-based
authentication only.

---

## Task 4 — Firewall Hardening (CIS Control 4)

### Install UFW
```bash
sudo apt install ufw -y
```

### Set default policies
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### Allow SSH first (important — do this before enabling!)
```bash
sudo ufw allow ssh
```

 **Critical:** Always run `ufw allow ssh` BEFORE `ufw enable`
or you will lock yourself out of the server!

### Enable firewall
```bash
sudo ufw enable
sudo ufw status verbose
```

### Block ping (ICMP)
```bash
sudo vim /etc/ufw/before.rules
# Change echo-request from ACCEPT to DROP
```

**Why:** Implements Zero Trust — deny everything by default,
allow only what is needed. Blocking ping hides your server
from basic reconnaissance.

---

## Task 5 — Brute Force Protection with Fail2Ban

### Install Fail2Ban
```bash
sudo apt install fail2ban -y
```

### Enable and start service
```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

**Why:** Automatically blocks IPs after repeated failed login
attempts — stops brute force attacks without manual intervention.

---

## Task 6 — Disable Unused Services (CIS Control 2)

### List all services
```bash
systemctl list-unit-files --type=service
```

### Disable unnecessary services
```bash
sudo systemctl disable <service>
sudo systemctl stop <service>
```

**Why:** Every running service is a potential attack vector.
Disabling unused services reduces your attack surface.

---

## Task 7 — Logging & Auditing (CIS Control 8)

### View authentication logs
```bash
sudo less /var/log/auth.log
```

### Install and enable auditd
```bash
sudo apt install auditd -y
sudo systemctl enable auditd
```

**Why:** Logs provide forensic capability — without them there
is no trace of attacker actions on your system.

---

## Task 8 — File Permission Hardening (CIS Control 3)

### Check sensitive file permissions
```bash
ls -l /etc/shadow
```

### Fix permissions
```bash
sudo chmod 640 /etc/shadow
```

**Why:** /etc/shadow contains password hashes — if exposed,
attackers can crack them offline.

---
## Key AppSec Lessons Learned
1. Always patch and update — outdated packages are low hanging fruit
2. Never allow root SSH login
3. Disable password authentication — use SSH keys only
4. Always allow SSH before enabling UFW — or you lock yourself out
5. Fail2Ban is essential on any internet facing server
6. Disable every service you don't need
7. Monitor auth logs regularly for suspicious activity
8. File permissions on sensitive files matter

## Common Errors & Fixes
| Error | Cause | Fix |
|-------|-------|-----|
| Locked out after UFW enable | SSH not allowed before enabling | Reboot instance and fix rules |
| SSH brute force attempts | Port 22 open to internet | Use Fail2Ban + restrict to your IP |
| Root login still works | sshd_config not reloaded | Run systemctl restart ssh |
| Weak passwords accepted | pwquality not configured | Install libpam-pwquality |

## Screenshots
