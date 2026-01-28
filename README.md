# <img src="GLASSYDASH/public/vite.svg" alt="GLASSYDASH" width="40" height="40" align="center"> GLASSYDASH

<div align="center">

[![Version](https://img.shields.io/badge/version-1.1.6-blue.svg)](https://github.com/yourusername/glassy-dash)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE.md)
[![Node](https://img.shields.io/badge/node-%3E%3D%2018.0-brightgreen.svg)](https://nodejs.org/)
[![Docker](https://img.shields.io/badge/docker-supported-blue.svg)](https://www.docker.com/)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/yourusername/glassy-dash/pulls)
[![Stars](https://img.shields.io/github/stars/yourusername/glassy-dash?style=social)](https://github.com/yourusername/glassy-dash/stargazers)

**A modern, feature-rich note-taking application with collaboration, AI integration, and advanced features.**

[Live Demo](https://demo.glassydash.io) • [Documentation](GLASSYDASH/docs/README.md) • [Quick Start](QUICKSTART.md) • [Support](SUPPORT.md)

</div>

---

## ✨ Features

GLASSYDASH is a powerful, private, and feature-rich note-taking application designed for individuals and teams.

### 🎯 Core Features

| Feature | Description |
|----------|-------------|
| **📝 Rich Note Editing** | Markdown support, formatting, drawing tools, checklists |
| **🤖 Private AI Assistant** | Local Llama 3.2 AI with RAG - 100% private |
| **👥 Real-time Collaboration** | Work together on notes simultaneously |
| **🔍 Advanced Search** | Full-text search with AI-powered question answering |
| **🏷️ Smart Tagging** | Organize notes with tags and filters |
| **✅ Interactive Checklists** | Task lists with drag-to-reorder |
| **🎨 Advanced Theming** | Custom backgrounds, accent colors, transparency levels |
| **💾 Import/Export** | Backup data, migrate from Google Keep |
| **🔐 Secure Auth** | JWT-based authentication with secret key recovery |
| **👨‍💼 Admin Panel** | User management and system monitoring |

### 🚀 Why GLASSYDASH?

- **🔒 Privacy First**: Your data stays on your server. No cloud dependency.
- **⚡ Lightning Fast**: Built with React 18 and Vite for optimal performance.
- **🎨 Beautiful UI**: Glassmorphism design with customizable themes.
- **📱 Responsive**: Works perfectly on desktop, tablet, and mobile.
- **🤖 AI-Powered**: Local AI understands your notes - no data leaves your device.
- **👥 Collaborate**: Real-time collaboration with teammates.
- **💻 Open Source**: MIT licensed - use it however you want.

---

## 🚀 Quick Start

Get GLASSYDASH running in under 5 minutes.

### ⚡ Docker (Recommended)

```bash
git clone https://github.com/yourusername/glassy-dash.git
cd glassy-dash/GLASSYDASH
docker-compose up -d
```

**Access**: http://localhost:8080  
**Default Login**: `admin` / `admin`

<details>
<summary><strong>💻 Development Setup</strong></summary>

```bash
git clone https://github.com/yourusername/glassy-dash.git
cd glassy-dash/GLASSYDASH
npm install
ADMIN_EMAILS=admin npm run dev
```

**Access**: http://localhost:5173

</details>

<details>
<summary><strong>📋 Manual Installation</strong></summary>

```bash
npm install
npm run build
npm start
```

</details>

> 📖 **Need more details?** Check our [Quick Start Guide](QUICKSTART.md)

---

## 📸 Screenshots

<div align="center">

**Dashboard (Light Mode)**
<br>
<img src="GLASSYDASH/public/dashboard-light.png" alt="Dashboard Light" width="800">

**Dashboard (Dark Mode)**
<br>
<img src="GLASSYDASH/public/dashboard-dark.png" alt="Dashboard Dark" width="800">

**AI Assistant**
<br>
<img src="GLASSYDASH/public/ai-assistant.png" alt="AI Assistant" width="800">

**Real-time Collaboration**
<br>
<img src="GLASSYDASH/public/collaboration.png" alt="Collaboration" width="800">

</div>

---

## 🛠 Technology Stack

### Frontend
| Technology | Purpose |
|------------|---------|
| [React 18](https://reactjs.org/) | UI Framework |
| [Vite 5](https://vitejs.dev/) | Build Tool & Dev Server |
| [Tailwind CSS 4](https://tailwindcss.com/) | Styling |
| [React Router 6](https://reactrouter.com/) | Client-side Routing |
| [TanStack Query](https://tanstack.com/query) | Data Fetching & Caching |
| [Lucide React](https://lucide.dev/) | Icons |
| [Marked](https://marked.js.org/) | Markdown Parsing |

### Backend
| Technology | Purpose |
|------------|---------|
| [Node.js 18+](https://nodejs.org/) | Runtime |
| [Express.js 4](https://expressjs.com/) | Web Framework |
| [better-sqlite3](https://github.com/WiseLibs/better-sqlite3) | Database |
| [JWT](https://jwt.io/) | Authentication |
| [Ollama](https://ollama.ai/) | Local Llama 3.2 AI |

### DevOps
| Technology | Purpose |
|------------|---------|
| [Docker](https://www.docker.com/) | Containerization |
| [Docker Compose](https://docs.docker.com/compose/) | Multi-container Orchestration |
| [GitHub Actions](https://github.com/features/actions) | CI/CD |

---

## 📚 Documentation

Comprehensive documentation is available for users, developers, and admins.

### 👤 For Users
- [Quick Start Guide](QUICKSTART.md) - Get started in 5 minutes
- [User Guide](GLASSYDASH/docs/user/GETTING_STARTED.md) - Learn all features
- [Features Overview](GLASSYDASH/docs/user/FEATURES.md) - Discover capabilities
- [FAQ](GLASSYDASH/docs/user/FAQ.md) - Common questions

### 💻 For Developers
- [Developer Setup](GLASSYDASH/docs/dev/SETUP.md) - Development environment
- [Architecture](GLASSYDASH/docs/dev/ARCHITECTURE.md) - System design
- [API Documentation](GLASSYDASH/docs/api/README.md) - API reference
- [Contributing Guide](CONTRIBUTING.md) - How to contribute

### 👨‍💼 For Admins
- [Admin Guide](GLASSYDASH/docs/admin/INSTALLATION.md) - Admin setup
- [Configuration](GLASSYDASH/docs/admin/CONFIGURATION.md) - System config
- [Security](GLASSYDASH/docs/admin/SECURITY.md) - Security best practices
- [Troubleshooting](GLASSYDASH/docs/admin/TROUBLESHOOTING.md) - Common issues

**[📖 Complete Documentation](GLASSYDASH/docs/README.md)**

---

## 🌟 Highlights

### 🤖 Private AI Assistant
GLASSYDASH includes a fully private AI assistant powered by Llama 3.2 (1B):

- **100% Local**: Runs entirely on your server. No data leaves your hardware.
- **Note-Aware**: Uses RAG to read your notes for accurate answers.
- **Smart Search**: Ask questions and get direct answers based on your data.
- **Privacy First**: Your notes never leave your device.

**Example Queries:**
- "What are my AWS commands?"
- "How old am I?"
- "Show me all notes about project X"

### 👥 Real-time Collaboration
Work together seamlessly:

- **Live Editing**: Multiple users can edit notes simultaneously
- **Real-time Sync**: Changes appear instantly for all collaborators
- **View-Only Mode**: Read without overwriting others' edits
- **Conflict Resolution**: Automatic resolution of concurrent edits

### 🎨 Advanced Theming
Customize your workspace:

- **Theme Presets**: One-click themes (Neon Tokyo, Zen Garden, etc.)
- **Custom Backgrounds**: High-quality library (Mobile/Desktop/4K optimized)
- **Accent Colors**: 7 bioluminescent options
- **Transparency**: 5 levels from airy to solid
- **Smart Overlay**: Ensures readability on any background

---

## 📦 Project Structure

```
glassy-dash/
├── GLASSYDASH/              # Main application
│   ├── src/                # React source code
│   │   ├── components/      # React components
│   │   ├── contexts/        # React contexts
│   │   ├── hooks/          # Custom hooks
│   │   ├── utils/          # Utility functions
│   │   └── App.jsx         # Main application
│   ├── server/             # Express.js backend
│   │   ├── routes/         # API routes
│   │   ├── middleware/     # Express middleware
│   │   └── index.js        # Server entry point
│   ├── docs/               # Comprehensive documentation
│   ├── public/             # Static assets
│   ├── tests/              # Test files
│   └── package.json
├── dashydash/               # Dashboard application
└── README.md                # This file
```

---

## 🔧 Configuration

### Environment Variables

Create a `.env` file in `GLASSYDASH/`:

```bash
# Application
NODE_ENV=production
API_PORT=8080

# Database
DB_FILE=/app/data/notes.db

# Security
JWT_SECRET=your-secret-key-here

# Admin
ADMIN_EMAILS=admin
ALLOW_REGISTRATION=false

# AI (Ollama)
OLLAMA_HOST=http://localhost:11434
OLLAMA_MODEL=llama3.2:1b
```

See `.env.example` in GLASSYDASH for all available options.

---

## 🚢 Docker Deployment

### Quick Start

```bash
cd GLASSYDASH
docker-compose up -d
```

### Production

```bash
docker-compose -f docker-compose.prod.yml up -d
```

### Docker Management

```bash
# View logs
docker-compose logs -f

# Stop services
docker-compose down

# Rebuild
docker-compose build --no-cache
```

> 📖 **Detailed Docker Guide**: [Docker Documentation](GLASSYDASH/docs/admin/INSTALLATION.md)

---

## 🧪 Testing

```bash
cd GLASSYDASH

# Unit tests
npm run test

# E2E tests
npm run test:e2e

# Linting
npm run lint
```

---

## 🤝 Contributing

We welcome contributions! Please follow these steps:

1. 📖 Read [Contributing Guidelines](CONTRIBUTING.md)
2. 🍴 Fork the repository
3. 🌿 Create a feature branch
4. ✨ Make your changes
5. 🧪 Add tests if applicable
6. 📝 Update documentation
7. 🚀 Submit a pull request

### Good First Issues
[![Good First Issues](https://img.shields.io/badge/good%20first%20issues-5-brightgreen.svg)](https://github.com/yourusername/glassy-dash/issues?q=is%3Aopen+is%3Aissue+label%3A%22good+first+issue%22)

Start with issues labeled [good first issue](https://github.com/yourusername/glassy-dash/issues?q=is%3Aopen+is%3Aissue+label%3A%22good+first+issue%22)!

---

## 📊 Comparison

| Feature | GLASSYDASH | Google Keep | Notion |
|----------|--------------|--------------|---------|
| Private AI | ✅ Local | ❌ Cloud | ❌ Cloud |
| Real-time Collab | ✅ | ❌ | ✅ |
| Self-Hosted | ✅ | ❌ | ❌ |
| Open Source | ✅ | ❌ | ❌ |
| Drawing Notes | ✅ | ❌ | ❌ |
| Advanced Theming | ✅ | ❌ | ✅ |
| Import/Export | ✅ | ✅ | ✅ |
| Free Forever | ✅ | ✅ | ❌ |
| Offline Mode | ✅ | ✅ | ✅ |

---

## 🗺️ Roadmap

### v0.68 (February 2026)
- [ ] Enhanced AI with larger models
- [ ] Improved mobile experience
- [ ] Advanced search filters

### v0.70 (March 2026)
- [ ] Plugin system
- [ ] Multi-language support
- [ ] Enhanced collaboration features

### v1.0.0 (Q2 2026)
- [ ] Mobile app (React Native)
- [ ] Enterprise features
- [ ] API v2 with breaking changes

[📝 Full Roadmap](https://github.com/yourusername/glassy-dash/milestones)

---

## 🏆 Sponsors

Thank you to all our sponsors! ❤️

[![Sponsors](https://opencollective.com/glassy-dash/tiers/sponsor/badge.svg?label=sponsors&width=600)](https://opencollective.com/glassy-dash)

### Diamond Sponsors
- [Your Company](https://github.com/sponsors) - Become a sponsor!

### Gold Sponsors
- [Your Name](https://github.com/sponsors) - Become a sponsor!

[Become a Sponsor](https://github.com/sponsors/yourusername) ❤️

---

## 📞 Support

- 🐛 [Report a Bug](https://github.com/yourusername/glassy-dash/issues/new?template=bug_report.md)
- 💡 [Request Feature](https://github.com/yourusername/glassy-dash/issues/new?template=feature_request.md)
- 💬 [GitHub Discussions](https://github.com/yourusername/glassy-dash/discussions)
- 📧 Email: [support@glassydash.io](mailto:support@glassydash.io)
- 💬 Discord: [Join our server](https://discord.gg/glassydash)

📖 [Support Guide](SUPPORT.md)

---

## 📜 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE.md) file for details.

---

## 🙏 Acknowledgments

- [React team](https://reactjs.org/) for the amazing framework
- [Vite team](https://vitejs.dev/) for the excellent build tool
- [Ollama](https://ollama.ai/) for local AI capabilities
- [The open-source community](https://github.com/yourusername/glassy-dash/graphs/contributors) for contributions

---

## 🔗 Links

- [🌐 Homepage](https://glassydash.io)
- [📚 Documentation](GLASSYDASH/docs/README.md)
- [🚀 Quick Start](QUICKSTART.md)
- [💬 Support](SUPPORT.md)
- [📝 Blog](https://glassydash.io/blog)
- [📢 Twitter/X](https://twitter.com/glassydash)
- [💬 Discord](https://discord.gg/glassydash)
- [📺 YouTube](https://youtube.com/@glassydash)

---

<div align="center">

**GLASSYDASH** - Modern note-taking, simplified.

Made with ❤️ by the GLASSYDASH Team

[⭐ Star us on GitHub](https://github.com/yourusername/glassy-dash) • [🐛 Report Issues](https://github.com/yourusername/glassy-dash/issues) • [💡 Feature Requests](https://github.com/yourusername/glassy-dash/issues)

</div>

---

**Version**: 1.1.6 | **Last Updated**: January 28, 2026
