---
description: How to deploy GlassyDash to the remote production VM
---

// turbo-all

# Remote Deployment Workflow

Follow these steps to deploy a new version of the application to the remote VM (`192.168.122.45`) through the jump host (`104.225.217.232`).

### 1. Build & Stage (Local)

Build the production image and save it as a tarball.

```bash
docker build -t glassy-dash:latest .
docker save glassy-dash:latest -o glassy-dash.tar
```

### 2. Transfer to Jump Host

Push the tarball and configuration scripts to the jump host.

```bash
scp glassy-dash.tar docker-compose.prod.yml docker_manage.sh glassy-jump:~/
```

### 3. Transfer to Target VM

Move the files from the jump host to the actual application VM.

```bash
ssh -t glassy-jump "scp ~/glassy-dash.tar ~/docker-compose.prod.yml ~/docker_manage.sh pozi@192.168.122.45:~/"
```

### 4. Deploy on VM

Load the image and restart the service using the management script.

```bash
ssh -t glassy-jump "ssh -t pozi@192.168.122.45 'sudo docker load -i ~/glassy-dash.tar && chmod +x ~/docker_manage.sh && sudo ./docker_manage.sh prod-compose'"
```

### 5. Verify Health

Check the logs and health endpoint.

```bash
ssh -t glassy-jump "ssh -t pozi@192.168.122.45 'sudo docker logs glassy-dash-prod --tail 20 && curl -f http://localhost:8080/api/monitoring/health'"
```

> [!IMPORTANT]
> Ensure the Cloudflare Tunnel is pointing to `http://localhost:3001` if it is running on the VM, or `http://192.168.122.45:3001` if it is running on the jump host.

---

## Unified Deploy Script (Recommended)

For a single-command deploy, use the `deploy.sh` script in the repo root:

```bash
./deploy.sh
```

This script handles all 5 steps above automatically. Use `--skip-build` to redeploy an existing tarball.
