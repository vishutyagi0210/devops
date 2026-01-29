# Docker Troubleshooting Runbook – High Memory Usage

---

## 📋 Overview

**Scenario:** System memory usage reaches critical level due to Docker containers.

**Target Service:** `docker` (dockerd)

**Purpose:** This runbook provides a systematic approach to diagnosing and resolving Docker memory issues through evidence-based troubleshooting.

---

## 🔍 Troubleshooting Steps

### 1. System Validation

#### Docker Service Status Check

```bash
systemctl status docker
docker --version
docker info
```

**Observation:**
Docker daemon is running and operational.

**Potential Issues:**
- Docker daemon not running
- Outdated Docker version
- Docker service crashes
- Configuration errors

**Immediate Fix:**
- Start Docker service: `systemctl start docker`
- Check Docker logs: `journalctl -u docker -n 100`
- Upgrade Docker to stable version
- Review daemon configuration in `/etc/docker/daemon.json`

---

### 2. Memory Usage Overview

```bash
free -h
docker stats --no-stream
ps aux --sort=-%mem | head -20
```

**Observation:**
Identify containers consuming excessive memory.

**Potential Issues:**
- Containers with memory leaks
- No memory limits configured
- Too many containers running
- System swap exhausted

**Immediate Fix:**
- Stop high-memory containers: `docker stop <container_id>`
- Set memory limits: `docker update --memory="512m" <container_id>`
- Remove unused containers: `docker container prune`
- Clear system cache: `sync; echo 3 > /proc/sys/vm/drop_caches`

---

### 3. Container Inspection

```bash
docker ps -a
docker stats --no-stream --format "table {{.Container}}\t{{.MemUsage}}\t{{.MemPerc}}"
docker inspect <container_id> | grep -i memory
```

**Observation:**
Detailed view of container resource consumption.

**Potential Issues:**
- Containers without memory limits
- Memory limit too high
- Container constantly restarting (OOM)
- Application memory leak

**Immediate Fix:**
- Set appropriate memory limits in docker-compose.yml or run command
- Restart problematic containers
- Update application code to fix memory leaks
- Enable swap memory for containers if needed

---

### 4. Docker System Resources

```bash
docker system df
docker system df -v
du -sh /var/lib/docker
```

**Observation:**
Check disk space used by Docker images, containers, and volumes.

**Potential Issues:**
- Excessive disk usage by Docker
- Too many unused images
- Large log files
- Orphaned volumes

**Immediate Fix:**
- Clean up unused resources: `docker system prune -a`
- Remove unused volumes: `docker volume prune`
- Configure log rotation in daemon.json
- Remove specific images: `docker rmi <image_id>`

---

### 5. Container Logs Review

```bash
docker logs --tail 100 <container_id>
docker inspect <container_id> | grep LogPath
ls -lh $(docker inspect --format='{{.LogPath}}' <container_id>)
```

**Observation:**
Check for errors, warnings, or excessive logging.

**Potential Issues:**
- Log files consuming excessive disk space
- Application errors causing memory issues
- No log rotation configured
- Verbose logging enabled

**Immediate Fix:**
- Truncate large log files: `truncate -s 0 <log_path>`
- Configure log rotation in daemon.json:
  ```json
  {
    "log-driver": "json-file",
    "log-opts": {
      "max-size": "10m",
      "max-file": "3"
    }
  }
  ```
- Restart Docker daemon after config change

---

### 6. Network and Volume Check

```bash
docker network ls
docker volume ls
docker inspect <container_id> | grep -A 10 Mounts
```

**Observation:**
Verify network and volume configuration.

**Potential Issues:**
- Orphaned networks
- Unused volumes consuming space
- Volume mounting issues
- Network connectivity problems

**Immediate Fix:**
- Remove unused networks: `docker network prune`
- Remove unused volumes: `docker volume prune`
- Check volume permissions
- Recreate networks if needed

---

### 7. Process and Kernel Check

```bash
dmesg | grep -i "out of memory"
dmesg | grep -i "killed process"
cat /var/log/syslog | grep -i "oom"
sysctl vm.overcommit_memory
```

**Observation:**
Check for OOM killer activity and kernel memory management.

**Potential Issues:**
- OOM killer terminating containers
- Kernel memory parameters not optimized
- System-wide memory pressure
- Swap disabled or insufficient

**Immediate Fix:**
- Enable swap: `swapon -a`
- Adjust overcommit settings: `sysctl -w vm.overcommit_memory=1`
- Add memory to the system
- Distribute containers across multiple hosts

---

### 8. Docker Daemon Configuration

```bash
cat /etc/docker/daemon.json
docker info | grep -i "log\|storage"
systemctl cat docker | grep ExecStart
```

**Observation:**
Review Docker daemon configuration for optimization.

**Potential Issues:**
- Inefficient storage driver
- No resource constraints configured
- Missing log rotation
- Suboptimal daemon settings

**Immediate Fix:**
- Configure daemon.json with proper limits:
  ```json
  {
    "default-address-pools": [
      {"base": "172.17.0.0/16", "size": 24}
    ],
    "storage-driver": "overlay2",
    "log-driver": "json-file",
    "log-opts": {
      "max-size": "10m",
      "max-file": "3"
    }
  }
  ```
- Restart Docker: `systemctl restart docker`

---

## 📊 Quick Findings

- ✅ Docker daemon operational
- ⚠️ High memory usage identified
- ⚠️ Missing memory limits on containers
- ⚠️ No log rotation configured

---

## 🚀 Escalation / Next Steps

If the issue persists after completing all troubleshooting steps:

1. **Enable Docker debug logging**
   ```bash
   # Edit daemon.json
   {
     "debug": true,
     "log-level": "debug"
   }
   # Restart Docker
   systemctl restart docker
   ```

2. **Monitor container metrics over time**
   ```bash
   docker stats
   # Or use monitoring tools like cAdvisor, Prometheus
   ```

3. **Review application architecture**
   - Evaluate if containers need to be split
   - Consider horizontal scaling
   - Review database connection pooling
   - Optimize application memory usage

4. **Consider orchestration solutions**
   - Migrate to Kubernetes for better resource management
   - Implement Docker Swarm for clustering
   - Use resource quotas and limits

---

## 🎓 Learning Outcome

This runbook helps in:

- **Evidence-based troubleshooting** – Making decisions based on actual system data
- **Identifying Docker memory patterns** – Recognizing common container memory issues
- **Applying safe corrective actions** – Implementing fixes that minimize service disruption
- **Proactive monitoring** – Setting up proper limits and logging to prevent future issues

---

## 📝 Additional Resources

- [Docker Documentation - Runtime Constraints](https://docs.docker.com/config/containers/resource_constraints/)
- [Docker Logging Configuration](https://docs.docker.com/config/containers/logging/configure/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Understanding Docker Memory Metrics](https://docs.docker.com/config/containers/runmetrics/)

---

## 🔧 Prevention Best Practices

### Always Set Resource Limits

```bash
docker run -d \
  --memory="512m" \
  --memory-swap="1g" \
  --cpus="1.5" \
  --name myapp \
  myapp:latest
```

### Use Docker Compose with Limits

```yaml
version: '3.8'
services:
  web:
    image: nginx
    deploy:
      resources:
        limits:
          memory: 256M
          cpus: '0.5'
        reservations:
          memory: 128M
          cpus: '0.25'
```

### Configure Log Rotation

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

### Regular Cleanup

```bash
# Weekly cleanup cron job
0 2 * * 0 docker system prune -af --volumes
```

---

## ⚠️ Important Notes

- Always backup volumes before pruning
- Test resource limits in development first
- Monitor container performance after applying limits
- Document all changes in change management system
- Coordinate with the team before stopping production containers
- Consider the impact on dependent services before restarting Docker daemon

---