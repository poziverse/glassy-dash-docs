# GlassyDash Deployment Lessons Learned

## Date: January 28, 2026
## Deployment Target: dash.0rel.com (VM via Cloudflare Tunnel)

---

## Executive Summary

Successfully deployed GlassyDash to production environment with Node.js backend, PM2 process manager, and Cloudflare tunnel for external access. This document captures key lessons, issues encountered, and best practices for future deployments.

---

## Environment Details

- **Host OS:** Linux (Debian/Ubuntu-based VM)
- **Node.js Version:** 20.20.0
- **Application Framework:** Express.js with React frontend
- **Process Manager:** PM2
- **External Access:** Cloudflare Tunnel (no exposed ports required)
- **Database:** SQLite3
- **Authentication:** JWT with bcrypt

---

## Critical Issues Encountered

### Issue 1: Application Returning "Cannot GET /"

**Symptom:**
- Application responded with HTTP 404 error for root path `/`
- API endpoints worked (`/api/auth/settings`), but static files were not served

**Root Cause:**
The server configuration only serves static files when `NODE_ENV === 'production'`:

```javascript
// server/index.js
if (NODE_ENV === 'production') {
  const dist = path.join(__dirname, '..', 'dist')
  app.use(express.static(dist))
  // SPA fallback - serve index.html for non-API routes
  app.use((req, res) => {
    if (req.path.startsWith('/api/')) {
      return res.status(404).json({ error: 'Not found' })
    }
    res.sendFile(path.join(dist, 'index.html'))
  })
}
```

The application was loading `.env` from `server/` directory which didn't exist, causing it to run in development mode (default).

**Solution:**
Copy the `.env` file from the project root to the `server/` directory:

```bash
cp ~/glassy-dash/.env ~/glassy-dash/server/.env
pm2 restart glassy-dash
```

**Lesson Learned:**
- Always verify the environment mode (development vs production) when static files aren't serving
- Check which `.env` file the application is actually loading
- Express apps often have different behavior between development and production modes

---

### Issue 2: Wrong Working Directory in PM2

**Symptom:**
- Application started but environment variables weren't loading correctly
- Logs showed it was loading from `server/.env` which didn't exist

**Root Cause:**
Initial PM2 command was run from within `server/` directory:
```bash
cd ~/glassy-dash/server
pm2 start node index.js --name glassy-dash
```

This caused dotenv to look for `.env` in the `server/` directory instead of project root.

**Solution:**
Start PM2 from the project root directory:
```bash
cd ~/glassy-dash
pm2 start server/index.js --name glassy-dash
```

**Lesson Learned:**
- Always check working directory when using PM2
- Relative paths in `server/index.js` are resolved from the directory where the app starts
- Document the correct startup directory in deployment documentation

---

### Issue 3: Port 3001 Not Externally Accessible (Initial Investigation)

**Symptom:**
- Application worked on `localhost:3001` on VM
- But curl from jump host (`192.168.122.45:3001`) timed out

**Root Cause:**
- Initially thought to be a firewall issue
- Investigated UFW and iptables rules
- Actually, the issue was that application wasn't in production mode yet

**Resolution:**
Fixed by setting `NODE_ENV=production` properly (see Issue 1)

**Lesson Learned:**
- Verify application is fully working locally before investigating external connectivity
- Check application logs for environment mode before assuming network issues
- Cloudflare tunnels can mask internal application issues since they're always "connected"

---

## Deployment Checklist

### Pre-Deployment

- [ ] Verify Node.js version compatibility (target: Node 20+)
- [ ] Install all production dependencies
- [ ] Build the application: `npm run build`
- [ ] Verify `dist/` directory contains all necessary files
- [ ] Prepare `.env` file with all required variables:
  - [ ] `PORT` (default: 3001)
  - [ ] `NODE_ENV` (must be `production` for static files)
  - [ ] `GEMINI_API_KEY` (for AI features)
  - [ ] `DATABASE_PATH` (SQLite database location)
  - [ ] `JWT_SECRET` (for authentication)

### Deployment Steps

1. **Clone Repository**
   ```bash
   git clone <repository-url> ~/glassy-dash
   cd ~/glassy-dash
   ```

2. **Install Dependencies**
   ```bash
   npm install --production
   ```

3. **Build Application**
   ```bash
   npm run build
   ```

4. **Configure Environment**
   ```bash
   # Create .env file with production settings
   cat > .env << EOF
   PORT=3001
   NODE_ENV=production
   GEMINI_API_KEY=your-key-here
   DATABASE_PATH=./data/glasskeep.db
   EOF
   
   # IMPORTANT: Copy to server directory
   cp .env server/.env
   ```

5. **Start with PM2**
   ```bash
   pm2 start server/index.js --name glassy-dash
   pm2 list
   ```

6. **Verify Deployment**
   ```bash
   # Check logs
   pm2 logs glassy-dash --lines 20
   
   # Test locally
   curl http://localhost:3001
   
   # Test API
   curl http://localhost:3001/api/auth/settings
   ```

7. **Configure Auto-Start**
   ```bash
   pm2 save
   pm2 startup
   # Run the provided command (usually requires sudo)
   ```

### Post-Deployment Verification

- [ ] Application responds on root path (HTML served)
- [ ] API endpoints respond correctly
- [ ] Static assets (CSS, JS) load properly
- [ ] Database operations work (create/update/delete notes)
- [ ] Authentication works (login/logout)
- [ ] AI features function (if using Gemini API)
- [ ] External URL accessible (dash.0rel.com)
- [ ] Monitor logs for errors in first 24 hours

---

## Best Practices

### 1. Environment Configuration

**Always set NODE_ENV explicitly:**
```bash
# .env file
NODE_ENV=production
PORT=3001
```

**Why:** Express and many other frameworks have different behavior in production vs development mode, especially regarding static file serving and error handling.

### 2. Process Management with PM2

**Use descriptive app names:**
```bash
pm2 start server/index.js --name glassy-dash-prod
```

**Save and configure auto-start:**
```bash
pm2 save
pm2 startup
```

**Why:** Ensures application survives server reboots and crashes automatically restart.

### 3. Working Directory Management

**Always verify the working directory:**
```bash
# Check current directory before starting
pwd
# Should be ~/glassy-dash, not ~/glassy-dash/server
```

**Why:** Relative paths in Node.js resolve based on where the process starts, not where the file is located.

### 4. Static File Serving

**Understand Express static middleware order:**
1. Define routes first (API endpoints)
2. Static file middleware second (`app.use(express.static(dist))`)
3. SPA fallback last (catch-all for non-API routes)

**Why:** Express processes middleware in order. If SPA fallback is defined before static files, you'll get "Cannot GET /" errors.

### 5. Logging and Monitoring

**Monitor PM2 regularly:**
```bash
pm2 list           # Check status
pm2 logs glassy-dash # View logs
pm2 monit           # Real-time monitoring
```

**Check application logs:**
```bash
# PM2 logs
/home/pozi/.pm2/logs/glassy-dash-out.log
/home/pozi/.pm2/logs/glassy-dash-error.log
```

**Why:** Early detection of issues prevents downtime and provides debugging information.

### 6. Cloudflare Tunnel Configuration

**Verify tunnel is running:**
```bash
ps aux | grep cloudflared
```

**Check tunnel logs:**
```bash
sudo journalctl -u cloudflared -f
```

**Why:** The tunnel is the only path to external access. If it's down, the site is down regardless of application status.

---

## Common Pitfalls to Avoid

### Pitfall 1: Assuming Development Mode Configuration Works in Production

**Mistake:** Deploying with default development settings

**Impact:** 
- Stack traces exposed to users
- Static files not served
- Performance issues (not optimized)

**Fix:** Always explicitly set `NODE_ENV=production` in `.env`

### Pitfall 2: Running from Wrong Directory

**Mistake:** Starting application from subdirectory without realizing it

**Impact:**
- `.env` file not found
- Relative paths resolve incorrectly
- Static file paths broken

**Fix:** Always check `pwd` before starting, and use absolute paths when in doubt

### Pitfall 3: Forgetting to Build

**Mistake:** Deploying source code without building

**Impact:**
- `dist/` directory doesn't exist
- Static files missing
- Application shows errors

**Fix:** Always run `npm run build` before starting in production

### Pitfall 4: Not Verifying After Changes

**Mistake:** Making configuration changes without testing

**Impact:**
- Downtime
- Configuration errors not discovered until users report them
- Harder to debug with live traffic

**Fix:** Test locally after every change before external verification

### Pitfall 5: Missing Environment Variables

**Mistake:** Deploying with incomplete `.env` file

**Impact:**
- Application uses defaults
- Features silently fail
- Security vulnerabilities (default secrets)

**Fix:** Create comprehensive `.env` file from `.env.example`

---

## Troubleshooting Guide

### Application Shows "Cannot GET /"

**Check:**
1. Is `NODE_ENV=production` set?
   ```bash
   grep NODE_ENV ~/.glassy-dash/.env
   ```

2. Is the `dist/` directory present?
   ```bash
   ls -la ~/glassy-dash/dist/
   ```

3. Restart application after environment changes
   ```bash
   pm2 restart glassy-dash
   ```

### API Works, Static Files Don't

**Check:**
1. Static middleware order in `server/index.js`
   - Routes should come before `express.static()`
   - SPA fallback should come last

2. Verify static file paths
   ```bash
   curl http://localhost:3001/assets/index-*.js
   ```

### Application Won't Start

**Check:**
1. Port already in use
   ```bash
   netstat -tlnp | grep 3001
   ```

2. Dependencies installed
   ```bash
   npm list --production
   ```

3. Database permissions
   ```bash
   ls -la ~/glassy-dash/data/
   ```

### PM2 Process Keeps Restarting

**Check:**
1. View error logs
   ```bash
   pm2 logs glassy-dash --err
   ```

2. Check memory limits
   ```bash
   pm2 show glassy-dash
   ```

3. Verify configuration
   ```bash
   cat ~/glassy-dash/server/.env
   ```

---

## Performance Optimization

### Reduce Memory Footprint

```bash
# Configure PM2 with memory limit
pm2 start server/index.js --name glassy-dash --max-memory-restart 500M
```

### Enable Clustering (if needed)

```bash
# Use all CPU cores
pm2 start server/index.js --name glassy-dash -i max
```

### Log Rotation

```bash
# Install logrotate module
pm2 install pm2-logrotate
pm2 set pm2-logrotate:max_size 10M
pm2 set pm2-logrotate:retain 7
```

---

## Security Considerations

### 1. Environment Variables

**Never commit `.env` files to git:**
```
# .gitignore
.env
server/.env
```

**Use different secrets for production vs development:**

### 2. API Rate Limiting

Application includes rate limiting:
```javascript
const rateLimit = require('express-rate-limit')
```

Monitor and adjust limits based on traffic patterns.

### 3. CORS Configuration

Current CORS setup allows all origins. For production:
```javascript
// Restrict to specific domains
app.use(cors({
  origin: ['https://dash.0rel.com', 'https://www.dash.0rel.com'],
  credentials: true
}))
```

### 4. Database Security

- Ensure database file is not web-accessible
- Regular backups of SQLite database
- File permissions: `600` (read/write for owner only)

---

## Maintenance Tasks

### Daily

- [ ] Check PM2 status: `pm2 list`
- [ ] Review error logs for critical issues
- [ ] Monitor application memory usage

### Weekly

- [ ] Review Cloudflare tunnel logs
- [ ] Check database file size
- [ ] Verify external access still works

### Monthly

- [ ] Update dependencies: `npm update`
- [ ] Security audit: `npm audit`
- [ ] Test backup restoration procedure

### Quarterly

- [ ] Node.js version update consideration
- [ ] Full security review
- [ ] Performance testing and optimization

---

## Contact and Support

### Internal Resources

- **Local Logs:** `/home/pozi/.pm2/logs/`
- **Application Logs:** PM2 logs for glassy-dash
- **System Logs:** `sudo journalctl -u cloudflared`

### Documentation

- **Deployment Guide:** `glassy-dash/DEPLOYMENT.md`
- **API Reference:** `glassy-dash/docs/API_REFERENCE.md`
- **Architecture:** `glassy-dash/docs/ARCHITECTURE.md`

### Quick Commands

```bash
# Restart application
pm2 restart glassy-dash

# View logs
pm2 logs glassy-dash --lines 100

# Check status
pm2 list

# Stop application
pm2 stop glassy-dash

# Start application
pm2 start glassy-dash

# Tunnel status
sudo systemctl status cloudflared
```

---

## Conclusion

This deployment was successful but highlighted several important lessons about Node.js application deployment, environment configuration, and the importance of verifying each layer of the stack. The key takeaway is to always:

1. **Verify the application environment mode** (development vs production)
2. **Check which .env file is being loaded** and from which directory
3. **Test locally before testing externally** to isolate application issues from network issues
4. **Understand the middleware order** in Express.js applications

By following this deployment guide and avoiding the common pitfalls listed above, future deployments should be smoother and more reliable.

---

**Document Version:** 1.0  
**Last Updated:** January 28, 2026  
**Deployment Environment:** dash.0rel.com (Production)  
**Application Version:** Latest from main branch