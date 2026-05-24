# 04 — Docker Setup

## Install Docker

```bash
sudo apt update
sudo apt install docker.io -y
```

---

## Start & Enable Docker

```bash
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker
```

---

## Add Users to Docker Group

This is critical — without this, Jenkins pipeline will fail with `permission denied` on Docker socket.

```bash
# Add jenkins user (REQUIRED for pipeline)
sudo usermod -aG docker jenkins

# Add ubuntu user (optional but useful)
sudo usermod -aG docker ubuntu

# Restart Jenkins to apply group change
sudo systemctl restart jenkins
```

Verify:
```bash
grep docker /etc/group
```

Expected output:
```
docker:x:999:jenkins,ubuntu
```

---

## Verify Docker Works

```bash
docker --version
sudo docker run hello-world
```

---

## How Docker Socket Works in Pipeline

```
EC2 Host
├── Docker Daemon (runs on host)
└── Docker Socket: /var/run/docker.sock
        ↓ mounted into
Maven Container (pipeline agent)
├── /var/run/docker.sock  ← talks to host daemon
└── docker CLI (installed in pipeline stage)
        ↓ builds images on
Host Docker Daemon ✅
```

> 💡 The socket mount (`-v /var/run/docker.sock:/var/run/docker.sock`) lets the container use the host's Docker daemon. But you still need the Docker CLI binary inside the container.

---

## Free Up Docker Space

If disk runs low:
```bash
# Remove all unused images, containers, volumes
sudo docker system prune -a -f

# Check disk after
df -h
```

---

## Common Errors

### Permission denied on docker socket
```
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock
```
**Fix:**
```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```
