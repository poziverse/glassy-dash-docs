# Deployment Guide (dash.0rel.com)

**Target Environment**: `glassy-vm` (192.168.122.45 via Jump Host)
**URL**: https://dash.0rel.com

## 1. Prerequisites

### Access

Ensure you have SSH access to the VM.
Run the setup script if keys are missing:

```bash
python3 deploy_access.py
```

### SSH Config

Ensure `~/.ssh/config` is configured:

```ssh
Host glassy-vm
    HostName 192.168.122.45
    User pozi
    ProxyJump poziverse@104.225.217.232
```

## 2. Deployment Steps

### Step 1: Prepare Local Build (Optional but recommended for testing)

```bash
cd GLASSYDASH
npm run build
```

### Step 2: Transfer Source Code

Transfer the source (excluding `node_modules`) to the VM.

```bash
# Create archive
tar --exclude='node_modules' --exclude='.git' --exclude='dist' -czf glassy-dash.tar.gz GLASSYDASH/

# Copy to VM
scp glassy-dash.tar.gz glassy-vm:~/
```

### Step 3: Remote Build & Restart

SSH into the VM and rebuild the Docker container.

```bash
ssh glassy-vm << 'EOF'
    # Clean old run
    rm -rf ~/app_source
    mkdir -p ~/app_source

    # Extract
    tar -xzf glassy-dash.tar.gz -C ~/app_source --strip-components=1

    # Build Docker Image
    cd ~/app_source
    sudo docker build -t glassy-dash:latest .

    # Restart Container (using verify script logic or manual)
    sudo docker rm -f GLASSYDASH || true

    sudo docker run -d \
        --name GLASSYDASH \
        --restart unless-stopped \
        -p 8080:8080 \
        -e NODE_ENV=production \
        -e API_PORT=8080 \
        -e JWT_SECRET=deployed-secret-key \
        -e DB_FILE=/app/data/notes.db \
        -e ADMIN_EMAILS=admin \
        -e ALLOW_REGISTRATION=true \
        -v ~/.GLASSYDASH:/app/data \
        glassy-dash:latest
EOF
```

### Step 4: Verification

Run the verification script:

```bash
python3 verify_deploy.py
```

Or check logs remotely:

```bash
ssh glassy-vm "sudo docker logs GLASSYDASH"
```
