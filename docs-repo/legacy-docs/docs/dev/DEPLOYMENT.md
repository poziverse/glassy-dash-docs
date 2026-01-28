# Deployment Guide

Complete guide to deploying GLASSYDASH to production environments.

---

## 🚀 Deployment Overview

GLASSYDASH can be deployed using multiple methods:

- **Docker**: Containerized deployment (recommended)
- **Node.js**: Direct Node.js deployment
- **Cloud Platforms**: Vercel, Railway, Render, etc.

**Architecture**: Single Node.js server with static frontend files

---

## 🐳 Docker Deployment

### Prerequisites

- Docker 20.10 or higher
- Docker Compose (optional)
- Domain name (optional)
- SSL certificate (optional)

### Quick Start

**Clone Repository**:
```bash
git clone https://github.com/poziverse/glassy-dash.git
cd glassy-dash
```

**Build Docker Image**:
```bash
docker build -t glassy-dash:latest .
```

**Run Container**:
```bash
docker run -d \
  --name glassy-dash \
  -p 3000:3000 \
  -v $(pwd)/data:/app/data \
  -e NODE_ENV=production \
  -e JWT_SECRET=your-secret-key \
  glassy-dash:latest
```

### Docker Compose

**docker-compose.yml**:
```yaml
version: '3.8'

services:
  glassy-dash:
    build: .
    container_name: glassy-dash
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - PORT=3000
      - JWT_SECRET=${JWT_SECRET}
      - OLLAMA_HOST=http://ollama:11434
    volumes:
      - ./data:/app/data
    restart: unless-stopped
    depends_on:
      - ollama

  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    ports:
      - "11434:11434"
    volumes:
      - ollama_data:/root/.ollama
    restart: unless-stopped

volumes:
  ollama_data:
```

**Deploy with Compose**:
```bash
# Create .env file
echo "JWT_SECRET=$(openssl rand -hex 32)" > .env

# Start services
docker-compose up -d

# View logs
docker-compose logs -f glassy-dash

# Stop services
docker-compose down
```

### Production Dockerfile

**Dockerfile**:
```dockerfile
# Stage 1: Build
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./
RUN npm ci --only=production

# Copy source code
COPY . .

# Build frontend
RUN npm run build

# Stage 2: Production
FROM node:20-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./
RUN npm ci --only=production

# Copy built frontend from builder
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/server ./server

# Create data directory
RUN mkdir -p /app/data

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# Start server
CMD ["node", "server/index.js"]
```

---

## 🖥️ Node.js Deployment

### Prerequisites

- Node.js 18.0.0 or higher
- npm or yarn
- PM2 (process manager)
- Nginx (reverse proxy, recommended)

### Manual Deployment

**1. Clone Repository**:
```bash
git clone https://github.com/poziverse/glassy-dash.git
cd glassy-dash
```

**2. Install Dependencies**:
```bash
npm install --production
```

**3. Build Frontend**:
```bash
npm run build
```

**4. Set Environment Variables**:
```bash
export NODE_ENV=production
export PORT=3000
export JWT_SECRET=$(openssl rand -hex 32)
export OLLAMA_HOST=http://localhost:11434
```

**5. Start Server**:
```bash
npm start
```

### PM2 Deployment

**Install PM2**:
```bash
npm install -g pm2
```

**Create ecosystem file** (ecosystem.config.js):
```javascript
module.exports = {
  apps: [{
    name: 'glassy-dash',
    script: './server/index.js',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 3000
    },
    env_production: {
      NODE_ENV: 'production',
      PORT: 3000,
      JWT_SECRET: process.env.JWT_SECRET,
      OLLAMA_HOST: process.env.OLLAMA_HOST
    },
    error_file: './logs/err.log',
    out_file: './logs/out.log',
    log_date_format: 'YYYY-MM-DD HH:mm:ss',
    merge_logs: true,
    max_memory_restart: '1G',
    autorestart: true,
    watch: false
  }]
};
```

**Start with PM2**:
```bash
# Start application
pm2 start ecosystem.config.js --env production

# Save PM2 configuration
pm2 save

# Setup PM2 startup script
pm2 startup

# View logs
pm2 logs glassy-dash

# Restart application
pm2 restart glassy-dash

# Stop application
pm2 stop glassy-dash
```

---

## 🔌 Nginx Configuration

### Basic Configuration

**/etc/nginx/sites-available/glassy-dash**:
```nginx
upstream glassy_dash {
    server localhost:3000;
}

server {
    listen 80;
    server_name your-domain.com;

    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com;

    # SSL certificates
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    # SSL settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Client body size limit
    client_max_body_size 10M;

    # Proxy settings
    location / {
        proxy_pass http://glassy_dash;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # WebSocket proxy
    location /ws {
        proxy_pass http://glassy_dash;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_read_timeout 86400;
        proxy_send_timeout 86400;
    }

    # Static files
    location /static {
        alias /path/to/glassy-dash/dist;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

**Enable Configuration**:
```bash
# Create symlink
sudo ln -s /etc/nginx/sites-available/glassy-dash /etc/nginx/sites-enabled/

# Test configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

---

## ☁️ Cloud Platform Deployment

### Railway

**Deploy with Railway**:

1. Connect GitHub repository
2. Add environment variables:
   - `NODE_ENV=production`
   - `JWT_SECRET` (generate random string)
   - `OLLAMA_HOST` (optional)

3. Deploy automatically on push

**Railway CLI**:
```bash
# Install Railway CLI
npm install -g @railway/cli

# Login
railway login

# Initialize project
railway init

# Set environment variables
railway variables set NODE_ENV=production
railway variables set JWT_SECRET=$(openssl rand -hex 32)

# Deploy
railway up
```

### Render

**Deploy with Render**:

1. Create new "Web Service"
2. Connect GitHub repository
3. Configure:
   - Build Command: `npm run build`
   - Start Command: `npm start`
   - Environment Variables: Add all required variables
4. Deploy

### Vercel

**Create vercel.json**:
```json
{
  "version": 2,
  "builds": [
    {
      "src": "server/index.js",
      "use": "@vercel/node"
    }
  ],
  "routes": [
    {
      "src": "/api/(.*)",
      "dest": "/server/index.js"
    },
    {
      "src": "/(.*)",
      "dest": "/dist/$1"
    }
  ],
  "env": {
    "NODE_ENV": "production"
  }
}
```

**Deploy with Vercel CLI**:
```bash
# Install Vercel CLI
npm install -g vercel

# Deploy
vercel --prod
```

---

## 🔒 Security Configuration

### Environment Variables

**Required Variables**:
```bash
NODE_ENV=production
PORT=3000
JWT_SECRET=your-super-secret-jwt-key-here
DATABASE_PATH=/app/data/notes.db
CORS_ORIGIN=https://your-domain.com
```

**Optional Variables**:
```bash
OLLAMA_HOST=http://localhost:11434
LOG_LEVEL=info
MAX_FILE_SIZE=10485760  # 10MB
RATE_LIMIT_WINDOW=900000  # 15 minutes
RATE_LIMIT_MAX=100
```

### Generate JWT Secret

```bash
# Linux/Mac
openssl rand -hex 32

# Node.js
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"

# Python
python3 -c "import secrets; print(secrets.token_hex(32))"
```

### SSL/TLS Setup

**Let's Encrypt (Certbot)**:
```bash
# Install Certbot
sudo apt-get install certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d your-domain.com

# Auto-renewal (automatic)
sudo certbot renew --dry-run
```

---

## 📊 Monitoring & Logging

### Application Logs

**PM2 Logs**:
```bash
# View all logs
pm2 logs

# View specific app logs
pm2 logs glassy-dash

# View last 100 lines
pm2 logs --lines 100

# Follow logs
pm2 logs --lines 0
```

**Docker Logs**:
```bash
# View logs
docker logs glassy-dash

# Follow logs
docker logs -f glassy-dash

# Last 100 lines
docker logs --tail 100 glassy-dash
```

### Health Checks

**Create health check endpoint** (if not exists):
```javascript
// server/routes/health.js
router.get('/health', (req, res) => {
  res.status(200).json({
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime()
  });
});
```

**Test health endpoint**:
```bash
curl http://localhost:3000/health
```

### Monitoring Tools

**PM2 Plus** (recommended):
```bash
# Install PM2 Plus
pm2 plus

# Connect to PM2 Plus dashboard
pm2 link <public_key> <secret_key>
```

**Uptime Monitoring**:
- UptimeRobot: https://uptimerobot.com/
- Pingdom: https://www.pingdom.com/
- StatusCake: https://www.statuscake.com/

---

## 🔄 Updates & Maintenance

### Update Application

**Docker**:
```bash
# Pull latest code
git pull origin main

# Rebuild image
docker build -t glassy-dash:latest .

# Stop and remove old container
docker stop glassy-dash
docker rm glassy-dash

# Start new container
docker run -d --name glassy-dash -p 3000:3000 \
  -v $(pwd)/data:/app/data \
  -e NODE_ENV=production \
  glassy-dash:latest
```

**PM2**:
```bash
# Pull latest code
git pull origin main

# Install dependencies
npm install --production

# Rebuild
npm run build

# Restart application
pm2 restart glassy-dash
```

**Docker Compose**:
```bash
# Pull latest code
git pull origin main

# Rebuild and restart
docker-compose up -d --build
```

### Database Backups

**Automated Backup Script**:
```bash
#!/bin/bash
# backup.sh

BACKUP_DIR="/backups"
DATE=$(date +%Y%m%d_%H%M%S)
DB_PATH="/app/data/notes.db"

mkdir -p $BACKUP_DIR

# Backup database
cp $DB_PATH $BACKUP_DIR/notes_$DATE.db

# Compress backup
gzip $BACKUP_DIR/notes_$DATE.db

# Keep only last 7 days of backups
find $BACKUP_DIR -name "notes_*.db.gz" -mtime +7 -delete

echo "Backup completed: notes_$DATE.db.gz"
```

**Setup Cron Job**:
```bash
# Edit crontab
crontab -e

# Add daily backup at 2 AM
0 2 * * * /path/to/backup.sh
```

### Database Migration

**Run Migrations**:
```bash
# Backup database first
./backup.sh

# Run migrations
npm run db:migrate

# Restart application
pm2 restart glassy-dash
```

---

## 🐛 Troubleshooting

### Common Issues

**Port Already in Use**:
```bash
# Find process
lsof -i :3000

# Kill process
kill -9 <PID>
```

**Database Locked**:
```bash
# Remove lock files
rm -f /app/data/notes.db-shm /app/data/notes.db-wal

# Restart application
pm2 restart glassy-dash
```

**Out of Memory**:
```bash
# Increase PM2 memory limit
pm2 restart glassy-dash --max-memory-restart 2G
```

**Application Won't Start**:
```bash
# Check logs
pm2 logs glassy-dash --lines 100

# Check environment variables
pm2 env glassy-dash

# Manually test
node server/index.js
```

### Performance Issues

**Check System Resources**:
```bash
# CPU usage
top

# Memory usage
free -h

# Disk usage
df -h
```

**Optimize Database**:
```bash
# VACUUM database
sqlite3 /app/data/notes.db "VACUUM;"

# Reindex
sqlite3 /app/data/notes.db "REINDEX;"
```

---

## 📝 Pre-Deployment Checklist

- [ ] Set up domain and DNS
- [ ] Generate JWT secret
- [ ] Configure environment variables
- [ ] Test database connection
- [ ] Set up SSL/TLS certificates
- [ ] Configure Nginx/reverse proxy
- [ ] Set up monitoring/logging
- [ ] Create backup strategy
- [ ] Test health endpoint
- [ ] Configure firewall rules
- [ ] Set up CI/CD pipeline
- [ ] Document deployment process
- [ ] Test rollback procedure

---

## 🎯 Production Best Practices

### Security

1. **Keep Dependencies Updated**:
   ```bash
   npm audit
   npm audit fix
   ```

2. **Use HTTPS Only**:
   - Force HTTPS redirect
   - Use strong SSL/TLS
   - Keep certificates updated

3. **Secure Headers**:
   - X-Frame-Options
   - X-Content-Type-Options
   - X-XSS-Protection
   - Content-Security-Policy

4. **Rate Limiting**:
   - Prevent brute force
   - Limit API requests
   - Monitor suspicious activity

### Performance

1. **Enable Compression**:
   ```nginx
   gzip on;
   gzip_types text/plain text/css application/json application/javascript;
   ```

2. **Cache Static Assets**:
   - Long cache headers
   - CDN for static files
   - Browser caching

3. **Database Optimization**:
   - Regular VACUUM
   - Proper indexes
   - Monitor query performance

### Reliability

1. **Process Management**:
   - Use PM2 or similar
   - Auto-restart on failure
   - Cluster mode for scaling

2. **Monitoring**:
   - Health checks
   - Error tracking
   - Performance metrics

3. **Backups**:
   - Automated daily backups
   - Off-site storage
   - Test restore procedure

---

## 📚 Resources

- **Setup Guide**: SETUP.md
- **Architecture Guide**: ARCHITECTURE.md
- **API Guide**: API.md
- **Testing Guide**: TESTING.md

---

**Last Updated**: January 23, 2026  
**GLASSYDASH Version**: 0.67 (Beta)