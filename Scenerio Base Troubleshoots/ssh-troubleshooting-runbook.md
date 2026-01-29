---

# 📄 File 3: `ssh-troubleshooting-runbook.md`

```markdown
# SSH Troubleshooting Runbook – Slow or Hanging Login
Author: <Your Name>  
Day 05 – Linux Troubleshooting Drill  

## Scenario
Users experience delays or hangs during SSH login.

## Target Service
sshd

---

## Environment Basics
```bash
uname -a
lsb_release -a
Observation
System stable and running Linux OS.

Potential Errors
DNS misconfiguration

Reverse DNS lookup delay

Immediate Fix
Update /etc/hosts

Disable reverse DNS lookup in sshd_config

Filesystem Sanity Check
mkdir /tmp/runbook-demo
cp /etc/hosts /tmp/runbook-demo/hosts-copy
ls -l /tmp/runbook-demo
Observation
Filesystem responsive.

Potential Errors
Disk full in /var/log

Immediate Fix
Rotate logs

Remove old log files

Snapshot: CPU & Memory
top
free -h
ps -o pid,pcpu,pmem,comm -C sshd
Observation
CPU spikes during login attempts.

Potential Errors
Brute force attacks

PAM misconfiguration

Immediate Fix
Block IPs

Enable fail2ban

Snapshot: Disk & IO
df -h
du -sh /var/log
vmstat 1 5
Observation
/var/log nearly full.

Potential Errors
Log flooding

Immediate Fix
Clean logs

Enable log rotation

Snapshot: Network
ss -tulpn | grep ssh
curl -I http://localhost
Observation
SSH listening on port 22.

Potential Errors
Firewall blocking SSH

Network latency

Immediate Fix
Open port 22

Restart networking

Logs Reviewed
journalctl -u ssh -n 50
tail -n 50 /var/log/auth.log
Observation
Repeated failed login attempts detected.

Potential Errors
Credential attacks

Broken automation scripts

Immediate Fix
Block IPs

Fix automation credentials

Quick Findings
SSH delay due to auth failures and disk pressure

Security concern identified

If This Worsens (Next Steps)
Enable SSH debug logging

Enforce key-based authentication

Notify security team

Learning Outcome
This runbook teaches:

SSH performance troubleshooting

Security awareness

Safe remediation steps

Resources
man sshd

man journalctl

Linux security guides