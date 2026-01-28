# Development Environment Setup

Complete guide to setting up a development environment for GLASSYDASH.

---

## 🚀 Prerequisites

### Required Software

**Node.js:**
- Version: 18.0.0 or higher
- Recommended: 20.x LTS
- Download: https://nodejs.org/

**Package Manager:**
- npm (comes with Node.js)
- yarn: `npm install -g yarn`
- pnpm: `npm install -g pnpm`

**Git:**
- Version: 2.23 or higher
- Download: https://git-scm.com/

### Optional Tools

**Code Editor:**
- VS Code (recommended): https://code.visualstudio.com/
- WebStorm: https://www.jetbrains.com/webstorm/
- Sublime Text: https://www.sublimetext.com/

**Browser:**
- Chrome DevTools (recommended)
- Firefox Developer Tools
- Edge DevTools

**Database Tools:**
- DB Browser for SQLite: https://sqlitebrowser.org/
- TablePlus: https://tableplus.com/

---

## 📦 Installation

### 1. Clone Repository

```bash
git clone https://github.com/poziverse/glassy-dash.git
cd glassy-dash
```

### 2. Install Dependencies

**Using npm:**
```bash
npm install
```

**Using yarn:**
```bash
yarn install
```

**Using pnpm:**
```bash
pnpm install
```

### 3. Set Up Environment Variables

Create a `.env` file in the project root:

```env
# Server Configuration
NODE_ENV=development
PORT=3000

# Database
DATABASE_PATH=./data/notes.db

# AI Configuration
OLLAMA_HOST=http://localhost:11434

# JWT Secret (generate a secure random string)
JWT_SECRET=your-super-secret-jwt-key-here

# CORS
CORS_ORIGIN=http://localhost:5173
```

**Generate JWT Secret:**
```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

### 4. Initialize Database

```bash
npm run init-db
```

Or manually:
```bash
node server/db.js init
```

---

## 🏗️ Project Structure

```
glassy-dash/
├── src/                    # Frontend source code
│   ├── components/         # React components
│   ├── contexts/           # React contexts
│   ├── hooks/              # Custom hooks
│   ├── lib/                # Utility functions
│   ├── providers/          # Context providers
│   ├── stores/             # State management
│   ├── utils/              # Helper functions
│   ├── App.jsx            # Main App component
│   ├── main.jsx           # Entry point
│   └── index.css          # Global styles
├── server/                 # Backend source code
│   ├── index.js            # Main server file
│   ├── db.js              # Database utilities
│   ├── routes/             # API routes
│   ├── middleware/         # Express middleware
│   ├── utils/              # Server utilities
│   └── migrations/         # Database migrations
├── public/                 # Static assets
├── data/                  # Database storage
├── docs/                  # Documentation
├── tests/                  # Test files
├── .github/                # GitHub workflows
├── package.json            # Dependencies
└── vite.config.js         # Vite configuration
```

---

## 🔧 Development Workflow

### Running Development Servers

**Start All Services:**
```bash
npm run dev
```

This starts:
- Frontend dev server (Vite) on port 5173
- Backend API server on port 3000
- Scheduler service

**Frontend Only:**
```bash
npm run dev:client
```

**Backend Only:**
```bash
npm run dev:server
```

### Code Quality Tools

**Linting:**
```bash
npm run lint
```

**Lint Fix:**
```bash
npm run lint:fix
```

**Format Code:**
```bash
npm run format
```

**Type Checking:**
```bash
npm run type-check
```

---

## 🧪 Testing

### Run All Tests

```bash
npm test
```

### Watch Mode

```bash
npm run test:watch
```

### Coverage Report

```bash
npm run test:coverage
```

### E2E Testing

```bash
npm run test:e2e
```

---

## 🔌 Database Management

### Reset Database

```bash
npm run db:reset
```

### Seed Database

```bash
npm run db:seed
```

### Run Migrations

```bash
npm run db:migrate
```

### View Database

```bash
sqlite3 data/notes.db
```

Or use DB Browser for SQLite.

---

## 🎨 Theming Development

### Adding New Themes

Edit `src/themes.js`:

```javascript
export const themes = {
  neonTokyo: {
    name: 'Neon Tokyo',
    colors: {
      primary: '#FF006E',
      secondary: '#8338EC',
      background: '#0F0F0F',
      // ...
    }
  },
  yourNewTheme: {
    name: 'Your Theme',
    colors: {
      primary: '#your-color',
      secondary: '#your-color',
      background: '#your-color',
      // ...
    }
  }
};
```

### Testing Themes

1. Add theme to `themes.js`
2. Restart dev server
3. Open Settings → Theme
4. Select your new theme
5. Verify colors and contrast

---

## 🤖 AI Development

### Setting Up Ollama

**Install Ollama:**
```bash
curl -fsSL https://ollama.ai/install.sh | sh
```

**Pull Model:**
```bash
ollama pull llama3.2:1b
```

**Verify Installation:**
```bash
ollama list
```

**Test Ollama:**
```bash
ollama run llama3.2:1b "Hello, how are you?"
```

### AI Integration Testing

1. Ensure Ollama is running
2. Set `OLLAMA_HOST` in `.env`
3. Restart server
4. Create a note
5. Use AI search feature
6. Verify responses

---

## 🐛 Debugging

### Frontend Debugging

**VS Code:**
1. Install "Debugger for Chrome" extension
2. Add `.vscode/launch.json`:
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "chrome",
      "request": "launch",
      "name": "Launch Chrome",
      "url": "http://localhost:5173",
      "webRoot": "${workspaceFolder}/src"
    }
  ]
}
```
3. Set breakpoints in VS Code
4. Press F5 to debug

### Backend Debugging

**VS Code:**
1. Add `.vscode/launch.json`:
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Launch Server",
      "program": "${workspaceFolder}/server/index.js",
      "env": {
        "NODE_ENV": "development"
      }
    }
  ]
}
```
2. Set breakpoints
3. Press F5 to debug

### Logging

**Frontend Logs:**
```javascript
console.log('Debug info:', data);
```

**Backend Logs:**
```javascript
logger.info('Server info:', message);
logger.error('Server error:', error);
```

**View Logs:**
- Browser console (F12)
- Server terminal
- `logs/server.log`

---

## 🔐 Security Development

### Testing Authentication

**Create Test User:**
```bash
node server/create_admin.js
```

**Test Login:**
```bash
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin"}'
```

**Verify JWT:**
```javascript
const token = localStorage.getItem('token');
console.log('Token:', token);
```

### Testing Permissions

1. Create user with "viewer" role
2. Try to edit a note
3. Verify access denied
4. Create user with "editor" role
5. Try to edit the same note
6. Verify access granted

---

## 📝 Common Issues

### Dependencies Installation Fails

**Solution:**
```bash
rm -rf node_modules
rm package-lock.json
npm install
```

### Database Locked

**Solution:**
```bash
rm -f data/notes.db-shm data/notes.db-wal
```

### Port Already in Use

**Find Process:**
```bash
lsof -i :3000
```

**Kill Process:**
```bash
kill -9 <PID>
```

### Ollama Not Responding

**Check Ollama Status:**
```bash
ps aux | grep ollama
```

**Restart Ollama:**
```bash
ollama serve
```

---

## 🚢 Production Build

### Build for Production

```bash
npm run build
```

### Preview Production Build

```bash
npm run preview
```

### Environment Variables for Production

```env
NODE_ENV=production
PORT=3000
CORS_ORIGIN=https://your-domain.com
JWT_SECRET=your-production-secret
```

---

## 📚 Next Steps

After setting up your development environment:

1. Read [Architecture Guide](ARCHITECTURE.md)
2. Review [Component Guide](../COMPONENT_GUIDE.md)
3. Check [API Reference](../API_REFERENCE.md)
4. Explore existing components in `src/components/`
5. Start making contributions!

---

## 🔗 Resources

- **Main README**: ../../README.md
- **Architecture**: ../ARCHITECTURE.md
- **API Reference**: ../API_REFERENCE.md
- **Testing Guide**: ../TESTING.md
- **Troubleshooting**: ../TROUBLESHOOTING.md

---

**Last Updated**: January 23, 2026  
**GLASSYDASH Version**: 0.67 (Beta)