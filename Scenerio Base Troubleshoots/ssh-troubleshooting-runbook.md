# SSH Troubleshooting Runbook – Slow or Hanging Login

**Author:** Your Name  
**Day:** 05 – Linux Troubleshooting Drill

---

## 📋 Overview

**Scenario:** Users experience delays or hangs during SSH login.

**Target Service:** `sshd`

**Purpose:** This runbook provides a systematic approach to diagnosing and resolving SSH login delays through evidence-based troubleshooting.

---

## 🔍 Troubleshooting Steps

### 1. System Validation

#### SSH Service Status Check

```bash
systemctl status sshd
ss -tlnp | grep :22
ps aux | grep sshd
```

**Observation:**
SSH daemon is running and listening on port 22.

**Potential Issues:**
- SSH service not running
- Port 22 not listening
- SSH daemon crashed
- Configuration syntax errors

**Immediate Fix:**
- Start SSH service: `systemctl start sshd`
- Check SSH logs: `journalctl -u sshd -n 100`
- Test configuration: `sshd -t`
- Restart SSH: `systemctl restart sshd`

---

### 2. DNS Resolution Check

```bash
# Test reverse DNS lookup
host <client_ip>
dig -x <client_ip>

# Check DNS settings
cat /etc/resolv.conf
cat /etc/nsswitch.conf | grep hosts
```

**Observation:**
DNS resolution delays are common causes of SSH hangs.

**Potential Issues:**
- Slow or failing DNS lookups
- DNS server unreachable
- Reverse DNS timeout
- Missing DNS configuration

**Immediate Fix:**
- Disable DNS lookup in SSH config:
  ```bash
  echo "UseDNS no" >> /etc/ssh/sshd_config
  systemctl restart sshd
  ```
- Add client IP to `/etc/hosts`
- Fix `/etc/resolv.conf` with valid nameservers
- Test DNS: `nslookup <hostname>`

---

### 3. SSH Configuration Review

```bash
cat /etc/ssh/sshd_config
sshd -T
grep -v "^#" /etc/ssh/sshd_config | grep -v "^$"
```

**Observation:**
Review SSH daemon configuration for performance issues.

**Potential Issues:**
- GSSAPI authentication enabled (slow)
- DNS lookup enabled
- Too many authentication methods
- Verbose logging causing delays
- PAM configuration issues

**Immediate Fix:**
- Disable GSSAPI:
  ```bash
  GSSAPIAuthentication no
  ```
- Disable DNS:
  ```bash
  UseDNS no
  ```
- Limit authentication methods:
  ```bash
  PubkeyAuthentication yes
  PasswordAuthentication yes
  ChallengeResponseAuthentication no
  ```
- Restart SSH: `systemctl restart sshd`

---

### 4. Authentication Logs Analysis

```bash
tail -n 100 /var/log/auth.log
journalctl -u sshd -n 100 --no-pager
grep "sshd" /var/log/secure
lastlog
```

**Observation:**
Identify authentication failures and delays.

**Potential Issues:**
- Failed login attempts causing delays
- PAM module timeouts
- Home directory issues
- Key permission problems
- Account lockouts

**Immediate Fix:**
- Check failed attempts: `grep "Failed password" /var/log/auth.log`
- Review PAM config: `cat /etc/pam.d/sshd`
- Fix home directory permissions: `chmod 755 /home/<user>`
- Check SSH key permissions: `chmod 600 ~/.ssh/authorized_keys`
- Clear failed login counters: `faillock --user <username> --reset`

---

### 5. Network and Firewall Check

```bash
# Check firewall rules
iptables -L -n -v | grep 22
ufw status
firewall-cmd --list-all

# Check network connectivity
netstat -an | grep :22
ss -tan | grep :22
tcpdump -i any port 22 -n
```

**Observation:**
Network issues can cause connection delays.

**Potential Issues:**
- Firewall blocking SSH
- Network latency
- TCP connection issues
- MTU problems
- Packet loss

**Immediate Fix:**
- Allow SSH through firewall: `ufw allow 22/tcp`
- Check MTU: `ip link show`
- Test connectivity: `ping <server_ip>`
- Check routes: `ip route`
- Disable TCP wrappers if causing issues

---

### 6. System Resource Check

```bash
uptime
free -h
df -h
top -bn1 | head -20
iostat -x 1 5
```

**Observation:**
System resource exhaustion can slow SSH logins.

**Potential Issues:**
- High system load
- Memory exhaustion
- Disk full
- High I/O wait
- CPU saturation

**Immediate Fix:**
- Kill resource-heavy processes
- Clear disk space: `du -sh /var/log/* | sort -h`
- Add swap if memory low
- Check for runaway processes: `ps aux --sort=-%cpu`
- Reboot if necessary

---

### 7. User Environment Check

```bash
# Check user shell and home directory
getent passwd <username>
ls -la /home/<username>
cat /home/<username>/.bashrc
cat /home/<username>/.bash_profile

# Check for slow profile scripts
time bash -c 'source ~/.bashrc'
```

**Observation:**
User environment scripts can cause login delays.

**Potential Issues:**
- Slow `.bashrc` or `.bash_profile`
- Network mounts in home directory
- Large history files
- Corrupted user profile
- Slow NFS mounts

**Immediate Fix:**
- Temporarily rename profile: `mv ~/.bashrc ~/.bashrc.bak`
- Check NFS mounts: `df -h | grep nfs`
- Clear large history: `> ~/.bash_history`
- Fix shell: `chsh -s /bin/bash <user>`
- Test with minimal profile

---

### 8. SSH Client-Side Debug

```bash
# Connect with verbose output
ssh -vvv user@host

# Test specific issues
ssh -o GSSAPIAuthentication=no user@host
ssh -o UseDNS=no user@host
ssh -o StrictHostKeyChecking=no user@host
```

**Observation:**
Client-side debugging shows where delays occur.

**Potential Issues:**
- Key exchange delays
- Authentication method negotiation
- DNS lookup hangs
- Host key verification slow
- Client configuration issues

**Immediate Fix:**
- Update client config `~/.ssh/config`:
  ```
  Host *
      GSSAPIAuthentication no
      ServerAliveInterval 60
      ServerAliveCountMax 3
      ConnectTimeout 10
  ```
- Clear known_hosts if needed
- Generate new SSH keys if corrupted

---

## 📊 Quick Findings

- ✅ SSH service running
- ⚠️ DNS lookups causing delays
- ⚠️ GSSAPI authentication enabled
- ⚠️ Slow user profile scripts detected

---

## 🚀 Escalation / Next Steps

If the issue persists after completing all troubleshooting steps:

1. **Enable SSH debug logging**
   ```bash
   # Edit /etc/ssh/sshd_config
   LogLevel DEBUG3
   # Restart SSH
   systemctl restart sshd
   # Monitor logs
   tail -f /var/log/auth.log
   ```

2. **Capture network traffic**
   ```bash
   tcpdump -i any port 22 -w ssh_debug.pcap
   # Analyze with wireshark
   ```

3. **Profile user login process**
   ```bash
   # Add timing to profile
   PS4='+ $(date "+%s.%N")\011 '
   bash -x ~/.bashrc 2>&1 | ts
   ```

4. **Check for security scanning**
   - Review fail2ban logs
   - Check for port scanning
   - Verify DenyHosts isn't blocking
   - Review SELinux/AppArmor denials

---

## 🎓 Learning Outcome

This runbook helps in:

- **Evidence-based troubleshooting** – Making decisions based on actual system data
- **Identifying SSH delay patterns** – Recognizing common causes of slow logins
- **Applying safe corrective actions** – Implementing fixes that maintain security
- **Optimizing SSH performance** – Configuring SSH for fast, reliable connections

---

## 📝 Additional Resources

- [OpenSSH Manual](https://www.openssh.com/manual.html)
- [SSH Performance Tuning](https://www.ssh.com/academy/ssh/performance)
- [Debugging SSH Issues](https://www.ssh.com/academy/ssh/troubleshooting)
- [PAM Configuration Guide](https://linux.die.net/man/5/pam.conf)

---

## 🔧 Prevention Best Practices

### Optimized SSH Configuration

```bash
# /etc/ssh/sshd_config
Port 22
Protocol 2
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication yes
UseDNS no
GSSAPIAuthentication no
MaxAuthTries 3
MaxStartups 10:30:60
LoginGraceTime 60
ClientAliveInterval 120
ClientAliveCountMax 3
```

### Client Configuration

```bash
# ~/.ssh/config
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    GSSAPIAuthentication no
    ConnectTimeout 10
    Compression yes
```

### Security Hardening

```bash
# Limit SSH access
AllowUsers user1 user2
AllowGroups sshusers

# Use key-based authentication
PasswordAuthentication no
PubkeyAuthentication yes

# Enable fail2ban
systemctl enable fail2ban
systemctl start fail2ban
```

### Monitoring Setup

```bash
# Add SSH monitoring
#!/bin/bash
# Check SSH response time
time ssh -o ConnectTimeout=5 localhost exit
if [ $? -ne 0 ]; then
    echo "SSH connection failed" | mail -s "SSH Alert" admin@example.com
fi
```

---

## 🔄 Runbook Maintenance

**Last Updated:** January 2026  
**Review Frequency:** Quarterly  
**Feedback:** Submit issues or improvements to the operations team

---

## ⚠️ Important Notes

- Always test SSH configuration before restarting: `sshd -t`
- Keep a backup session open when modifying SSH config
- Document all configuration changes
- Test changes from multiple networks
- Consider security implications of any performance optimizations
- Never disable security features without proper risk assessment

---

## 📈 Monitoring Checklist

- [ ] Set up SSH connection time alerts
- [ ] Monitor failed login attempts
- [ ] Track authentication method usage
- [ ] Log DNS resolution times
- [ ] Monitor system resources during peak hours
- [ ] Set up automated SSH health checks
- [ ] Create dashboards for SSH metrics

---

## 🐛 Common Issues Quick Reference

| Symptom | Likely Cause | Quick Fix |
|---------|--------------|-----------|
| 30-60s delay before password prompt | DNS lookup timeout | Set `UseDNS no` |
| Delay before password prompt | GSSAPI authentication | Set `GSSAPIAuthentication no` |
| Delay after password entry | Slow PAM modules | Review `/etc/pam.d/sshd` |
| Delay after successful login | Slow `.bashrc` or `.bash_profile` | Profile and optimize scripts |
| Connection timeout | Firewall blocking | Check `iptables` and `ufw` |
| Permission denied | Key permissions wrong | Fix permissions: `chmod 600 ~/.ssh/*` |
| Connection refused | SSH daemon not running | Start service: `systemctl start sshd` |