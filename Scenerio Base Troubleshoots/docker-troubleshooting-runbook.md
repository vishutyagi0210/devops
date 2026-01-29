---

# 📄 File 2: `docker-troubleshooting-runbook.md`

```markdown
# Docker Troubleshooting Runbook – High Memory Usage
Author: <Your Name>  
Day 05 – Linux Troubleshooting Drill  

## Scenario
System memory usage reaches critical level due to Docker containers.

## Target Service
docker (dockerd)

---

## Environment Basics
```bash
uname -a
cat /etc/os-release
Observation
Linux host running Docker Engine.

Potential Errors
cgroups disabled

Docker version outdated

Kernel not supporting memory limits

Immediate Fix
Restart Docker

Enable cgroups

Upgrade Docker

Filesystem Sanity Check
mkdir /tmp/runbook-demo
cp /etc/hosts /tmp/runbook-demo/hosts-copy
ls -l /tmp/runbook-demo
Observation
Filesystem writable.

Potential Errors
Disk full in /var/lib/docker

Immediate Fix
Run docker system prune -f

Snapshot: CPU & Memory
top
free -h
ps -o pid,pcpu,pmem,comm -C dockerd
docker stats --no-stream
Observation
One container consuming excessive memory.

Potential Errors
Application memory leak

No container memory limits

Swap thrashing

Immediate Fix
Restart problematic container

Apply memory limits

Reduce load

Snapshot: Disk & IO
df -h
du -sh /var/lib/docker
vmstat 1 5
Observation
Docker storage growing rapidly.

Potential Errors
Logs not rotated

Orphaned images

Immediate Fix
Enable log rotation

Remove unused images

Snapshot: Network
ss -tulpn | grep docker
curl --unix-socket /var/run/docker.sock http://localhost/info
Observation
Docker API responsive.

Potential Errors
API timeout

Containers unreachable

Immediate Fix
Restart docker service

Restart container networking

Logs Reviewed
journalctl -u docker -n 50
tail -n 50 /var/log/syslog
Observation
Frequent container restarts observed.

Potential Errors
OOM killed containers

Image pull failures

Immediate Fix
Increase memory limits

Fix registry connectivity

Quick Findings
Root cause likely memory leak

Disk usage increasing

If This Worsens (Next Steps)
Capture container heap dump

Add monitoring alerts

Migrate workload to larger node

Learning Outcome
This runbook teaches:

Container resource troubleshooting

Memory leak identification

Safe remediation strategies

Resources
Docker documentation

man free

man vmstat