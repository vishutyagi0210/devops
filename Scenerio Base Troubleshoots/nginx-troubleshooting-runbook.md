# Nginx Troubleshooting Runbook – Performance Degradation
Author: <Your Name>  
Day 05 – Linux Troubleshooting Drill  

---

## Scenario
Users report that the web application is slow or returning timeout errors.

## Target Service
nginx

---

## 1. System Validation

### OS & Kernel Check
```bash
uname -a
lsb_release -a
Observation
System is running a stable Linux distribution with a supported kernel.

Potential Issues

Kernel mismatch after system update

Unsupported or outdated nginx version

Pending reboot after kernel upgrade

Immediate Fix

Verify nginx version compatibility

Reboot system if kernel upgrade is pending

Upgrade or downgrade nginx to a supported version

2. Filesystem Sanity Check
mkdir /tmp/runbook-demo
cp /etc/hosts /tmp/runbook-demo/hosts-copy
ls -l /tmp/runbook-demo
Observation
Filesystem is writable and responsive.

Potential Issues

Disk full (No space left on device)

Permission denied

Read-only filesystem

Immediate Fix

Clean /var/log and /tmp

Remove large unused files

Fix directory permissions

3. CPU & Memory Snapshot
top
free -h
ps -o pid,pcpu,pmem,comm -C nginx
Observation
nginx shows moderate CPU usage and stable memory.

Potential Issues

CPU pegged at 100%

Memory leak in worker processes

OOM killer terminating nginx

Immediate Fix

Reload nginx service

Reduce worker processes

Restart nginx if necessary

4. Disk & I/O Snapshot
df -h
du -sh /var/log
vmstat 1 5
Observation
Disk usage below threshold and I/O wait is low.

Potential Issues

Disk 100% full

High I/O wait (>20%)

Log files growing uncontrollably

Immediate Fix

Run logrotate

Clear large log files

Investigate slow storage

5. Network Check
ss -tulpn | grep nginx
curl -I http://localhost
Observation
nginx is listening on port 80 and returning HTTP 200.

Potential Issues

Port not listening

Connection refused

Firewall blocking traffic

Immediate Fix

Restart nginx

Check firewall rules

Verify backend service connectivity

6. Log Review
journalctl -u nginx -n 50
tail -n 50 /var/log/nginx/error.log
Observation
Slow request warnings detected.

Potential Issues

502 / 504 gateway errors

Worker timeout errors

Permission denied accessing files

Immediate Fix

Restart nginx

Fix file ownership

Check upstream service health

7. Quick Findings
nginx operational but slow

Backend latency suspected

No critical system failure

8. Escalation / Next Steps
Enable nginx debug logging

Attach strace to nginx PID

Review backend services

9. Learning Outcome
This runbook helps in:

Evidence-based troubleshooting

Identifying nginx failure patterns

Applying safe corrective actions