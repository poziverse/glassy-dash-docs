# 🚀 Quick Start Guide

Get GLASSYDASH up and running in under 5 minutes.

---

## ⚡ Fastest Way: Docker (Recommended)

```bash
# Clone the repository
git clone https://github.com/yourusername/glassy-dash.git
cd glassy-dash/GLASSYDASH

# Run with Docker Compose
docker-compose up -d

# Access your application
# Open http://localhost:8080 in your browser
```

**That's it!** GLASSYDASH is now running with all features enabled.

### First Login

- **Username**: `admin`
- **Password**: `admin`

> ⚠️ **Important**: Change the default password after your first login via the Admin Panel.

---

## 📋 Docker Compose (Production Ready)

```bash
# Edit docker-compose.yml with your settings
# Then start
docker-compose -f docker-compose.prod.yml up -d

# View logs
docker-compose logs -f

# Stop
docker-compose down
```

---

## 💻 Development Setup

```bash
# Clone the repository
git clone https://github.com/yourusername/glassy-dash.git
cd glassy-dash/GLASSYDASH

# Install dependencies
npm install

# Start development server
ADMIN_EMAILS=admin npm run dev

# Open http://localhost:5173
```

### Development Features
- 🔄 Hot module replacement
- 🐛 Debug mode enabled
- 📊 API running on http://localhost:8080
- 🔧 File watching for changes

---

## 🔧 Manual Installation

### Prerequisites
- Node.js 18+ and npm
- (Optional) Docker and Docker Compose

### Steps

1. **Install Dependencies**
   ```bash
   npm install
   ```

2. **Configure Environment**
   ```bash
   cp .env.example .env
   # Edit .env with your settings
   ```

3. **Start the Server**
   ```bash
   npm run dev        # Development
   npm run build      # Build for production
   npm start          # Start production server
   ```

---

## ✅ Verify Installation

1. **Check the Application**
   - Open http://localhost:8080 (or 5173 for dev)
   - You should see the GLASSYDASH login screen

2. **Test Features**
   - Login with admin/admin
   - Create your first note
   - Try the AI assistant
   - Upload an image

3. **Check Logs**
   ```bash
   # Docker
   docker-compose logs -f
   
   # Development
   tail -f GLASSYDASH/server/server.log
   ```

---

## 🔑 Default Credentials

| Role | Username | Password |
|------|----------|----------|
| Admin | admin | admin |

> 🔒 **Security**: Change these credentials immediately after first login!

---

## 📱 What's Next?

### For Users
- [Read the User Guide](GLASSYDASH/docs/user/GETTING_STARTED.md) - Learn all features
- [Features Overview](GLASSYDASH/docs/user/FEATURES.md) - Discover capabilities
- [FAQ](GLASSYDASH/docs/user/FAQ.md) - Common questions

### For Developers
- [Developer Setup](GLASSYDASH/docs/dev/SETUP.md) - Development environment
- [Architecture](GLASSYDASH/docs/dev/ARCHITECTURE.md) - System design
- [API Documentation](GLASSYDASH/docs/api/README.md) - API reference

### For Admins
- [Admin Guide](GLASSYDASH/docs/admin/INSTALLATION.md) - Admin setup
- [Configuration](GLASSYDASH/docs/admin/CONFIGURATION.md) - System config
- [Security](GLASSYDASH/docs/admin/SECURITY.md) - Security best practices

---

## 🆘 Troubleshooting

### Port Already in Use
```bash
# Change port in .env or docker-compose.yml
API_PORT=8081
```

### Database Locked (SQLite)
```bash
# Stop all processes using the database
# Docker:
docker-compose down

# Development:
# Ctrl+C to stop server
```

### Connection Refused
```bash
# Verify API is running
curl http://localhost:8080/api/health

# Check logs for errors
docker-compose logs
```

### Need Help?
- 📖 [Troubleshooting Guide](GLASSYDASH/docs/admin/TROUBLESHOOTING.md)
- 💬 [GitHub Discussions](https://github.com/yourusername/glassy-dash/discussions)
- 🐛 [Report an Issue](https://github.com/yourusername/glassy-dash/issues)

---

## 🌟 Quick Tips

- 🎨 **Themes**: Go to Settings → Theme to customize colors and backgrounds
- 🔍 **Search**: Press `Ctrl+K` (or `Cmd+K`) for quick search
- 🤖 **AI**: Type questions in the search bar for AI assistance
- 📌 **Pinning**: Pin important notes for easy access
- 👥 **Collaborate**: Share notes with teammates via the collaboration menu
- 💾 **Backup**: Use ⋮ → Export to backup your notes

---

## 📚 More Documentation

- [Complete Documentation](GLASSYDASH/docs/README.md) - Full documentation index
- [Installation Guide](GLASSYDASH/docs/user/INSTALLATION.md) - Detailed setup
- [Support](SUPPORT.md) - Getting help

---

**GLASSYDASH v1.1.6** | [Homepage](https://github.com/yourusername/glassy-dash) | [Report Issue](https://github.com/yourusername/glassy-dash/issues)
