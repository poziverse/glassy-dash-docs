# GLASSYDASH Architecture

Comprehensive overview of GLASSYDASH's system architecture, design patterns, and technical decisions.

---

## 📐 Overview

GLASSYDASH is a **modern full-stack web application** built with:

- **Frontend**: React + Vite + TypeScript
- **Backend**: Node.js + Express
- **Database**: SQLite
- **Real-time**: WebSocket
- **AI**: Llama 3.2 via Ollama

**Architecture Pattern**: Client-Server with REST API

---

## 🎯 Design Principles

### Core Principles

1. **Privacy First**: All data stored locally, no cloud transmission
2. **Simplicity**: Minimal dependencies, clear code structure
3. **Performance**: Optimized for speed and efficiency
4. **Extensibility**: Easy to add new features
5. **Security**: JWT authentication, role-based access control

### Tech Stack Rationale

**React + Vite**:
- Fast HMR (Hot Module Replacement)
- Modern build tooling
- Excellent TypeScript support
- Large ecosystem

**Express.js**:
- Minimal and flexible
- Easy to test
- Good documentation
- Large middleware ecosystem

**SQLite**:
- Zero configuration
- Fast for small to medium datasets
- Single file database
- Perfect for self-hosting

**WebSocket**:
- Real-time collaboration
- Low overhead
- Native browser support
- Simple implementation

---

## 🏗️ System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   Client Browser                        │
│  ┌──────────────────────────────────────────────┐  │
│  │          React Frontend (Vite)             │  │
│  │  - Components (Shell Architecture)          │  │
│  │  - Zustand Stores (State Management)        │  │
│  │  - Hooks                                 │  │
│  │  - Utils                                 │  │
│  └──────────────────────────────────────────────┘  │
│           │ HTTP / SSE                          │
│           ▼                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │      Express.js Server                      │  │
│  │  - REST API Routes                       │  │
│  │  - SSE Server (Real-time Events)          │  │
│  │  - Middleware (Auth, Security, Logging)    │  │
│  └──────────────────────────────────────────────┘  │
│           │                                   │
│           ▼                                   │
│  ┌──────────────────────────────────────────────┐  │
│  │        SQLite Database                      │  │
│  │  - Notes, Users, Tags, Checklist Items   │  │
│  │  - Collaborators, Images                 │  │
│  └──────────────────────────────────────────────┘  │
│                                                 │
│           ┌───────────────────────────────────────┐  │
│           │      Ollama AI Service              │  │
│           │  - Llama 3.2 Model                │  │
│           └───────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

---

## 🔌 Frontend Architecture

### Component Hierarchy & Navigation

The application uses a **Single Page Application (SPA) Shell** pattern. The root `App.jsx` persists the `DashboardLayout` across route changes, ensuring that common UI elements like the sidebar and header do not re-render.

```
App.jsx (Router & Shell)
└── DashboardLayout (Persistent Shell)
    ├── Sidebar (Navigation & Tag Filter)
    ├── TopBar (Title, Search, AI Toggle)
    └── Main Content (Swappable Views)
        ├── NotesView (Default)
        ├── AdminView
        ├── SettingsView
        ├── TagsView
        ├── TrashView
        ├── DocsView
        └── VoiceWorkspace
```

### State Management

**Zustand Stores**:
The application has migrated from React Context to **Zustand** for better performance and simpler state access.

- **`uiStore`**: Manages sidebar state, page titles, and global UI actions.
- **`notesStore`**: Manages note data, filtering, and real-time synchronization.
- **`authStore`**: Handles user authentication and persistence.
- **`settingsStore`**: Manages theming and UI preferences.
- **`aiStore`**: Tracks AI assistant state and history.

---

## 🔄 Real-time Architecture

### Server-Sent Events (SSE) Implementation

Real-time updates are handled via **SSE** instead of WebSockets for better efficiency and simplicity over HTTP/1.1 and HTTP/2.

**Server-Side**:
The server maintains a stream of events at `/api/events`. When a note is updated, it broadcasts a `note_updated` event to all relevant clients.

**Client-Side**:
The `useCollaboration` hook (used by `notesStore`) listens for events and triggers selective data re-fetching or optimistic UI updates.

### API Structure

```
server/index.js
├── Express App Setup
├── Middleware
│   ├── auth.js - JWT verification
│   ├── cors.js - CORS configuration
│   ├── errorHandler.js - Global error handling
│   └── logging.js - Request logging
├── Routes
│   ├── auth.js - /api/auth/*
│   ├── notes.js - /api/notes/*
│   ├── users.js - /api/users/*
│   └── collaboration.js - /api/collaboration/*
├── WebSocket Server
│   ├── Connection management
│   ├── Real-time updates
│   └── Broadcast system
└── Database
    └── db.js - SQLite operations
```

### Middleware Pipeline

```
Request → CORS → Logging → Auth → Routes → Error Handler
                                                │
                                                ▼
                                            Response
```

**Auth Middleware**:
```javascript
const authMiddleware = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }
  
  try {
    const decoded = jwt.verify(token, JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
};
```

### Route Structure

**REST API Endpoints**:

```
/api
├── /auth
│   ├── POST   /login
│   ├── POST   /register
│   ├── POST   /logout
│   └── GET    /me
├── /notes
│   ├── GET    /          - List all notes
│   ├── GET    /:id        - Get single note
│   ├── POST   /          - Create note
│   ├── PUT    /:id        - Update note
│   ├── DELETE /:id        - Delete note
│   └── POST   /search     - Search notes
├── /users
│   ├── GET    /          - List users
│   ├── GET    /:id        - Get user
│   ├── POST   /          - Create user
│   ├── PUT    /:id        - Update user
│   └── DELETE /:id        - Delete user
└── /collaboration
    ├── POST   /:noteId/join    - Join collaboration
    ├── POST   /:noteId/leave   - Leave collaboration
    └── WebSocket / - Real-time updates
```

---

## 💾 Database Architecture

### Schema Design

**Users Table**:
```sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  username TEXT UNIQUE NOT NULL,
  password TEXT NOT NULL,
  email TEXT UNIQUE,
  role TEXT DEFAULT 'user',
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

**Notes Table**:
```sql
CREATE TABLE notes (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER NOT NULL,
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  type TEXT DEFAULT 'rich',
  tags TEXT, -- JSON array
  pinned INTEGER DEFAULT 0,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

**Tags Table**:
```sql
CREATE TABLE tags (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT UNIQUE NOT NULL,
  user_id INTEGER NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

**Collaborators Table**:
```sql
CREATE TABLE collaborators (
  note_id INTEGER NOT NULL,
  user_id INTEGER NOT NULL,
  permission TEXT NOT NULL, -- 'read' or 'write'
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (note_id, user_id),
  FOREIGN KEY (note_id) REFERENCES notes(id),
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

### Indexes

```sql
CREATE INDEX idx_notes_user_id ON notes(user_id);
CREATE INDEX idx_notes_created_at ON notes(created_at DESC);
CREATE INDEX idx_notes_pinned ON notes(pinned DESC);
CREATE INDEX idx_notes_tags ON notes(tags);
CREATE INDEX idx_tags_user_id ON tags(user_id);
```

### Database Operations

**Connection Pooling**:
```javascript
// server/db.js
const db = new sqlite3.Database(process.env.DATABASE_PATH || './data/notes.db', {
  verbose: process.env.NODE_ENV === 'development'
});

db.serialize(() => {
  // All database operations serialized
});
```

**Migration System**:
```javascript
// server/migrations/
exports.up = (db) => {
  db.run(`CREATE TABLE IF NOT EXISTS ...`);
};

exports.down = (db) => {
  db.run(`DROP TABLE IF EXISTS ...`);
};
```

---

## 🔄 Real-time Architecture

### WebSocket Implementation

**Server-Side**:
```javascript
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', (ws, req) => {
  const userId = getUserIdFromToken(req);
  
  ws.on('message', (message) => {
    const data = JSON.parse(message);
    
    // Broadcast to all connected clients
    broadcastUpdate({
      type: 'note_updated',
      noteId: data.noteId,
      userId: userId
    });
  });
});

function broadcastUpdate(data) {
  wss.clients.forEach(client => {
    if (client.readyState === WebSocket.OPEN) {
      client.send(JSON.stringify(data));
    }
  });
}
```

**Client-Side**:
```javascript
const ws = new WebSocket('ws://localhost:8080');

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  
  if (data.type === 'note_updated') {
    // Update local state
    updateNoteInState(data.noteId);
  }
};
```

### Collaboration Flow

```
User A edits note → WebSocket → Broadcast → User B sees update
                    │
                    ▼
               Update database
```

---

## 🤖 AI Integration Architecture

### RAG (Retrieval-Augmented Generation)

```
User Query
    │
    ▼
┌─────────────────────────┐
│  Vector Search        │
│  (Semantic Search)    │
└─────────────────────────┘
    │
    ▼
┌─────────────────────────┐
│  Context Retrieval   │
│  (Fetch Notes)        │
└─────────────────────────┘
    │
    ▼
┌─────────────────────────┐
│  Llama 3.2          │
│  (Generation)         │
└─────────────────────────┘
    │
    ▼
Response with Sources
```

### AI Service Flow

```javascript
// server/ai.js
async function aiQuery(query) {
  // 1. Search for relevant notes
  const relevantNotes = await vectorSearch(query);
  
  // 2. Build context
  const context = relevantNotes.map(note => 
    `- ${note.title}: ${note.content.slice(0, 100)}`
  ).join('\n');
  
  // 3. Query Ollama
  const response = await fetch(`${OLLAMA_HOST}/api/generate`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      model: 'llama3.2:1b',
      prompt: `Context:\n${context}\n\nQuestion: ${query}`,
      stream: false
    })
  });
  
  return await response.json();
}
```

---

## 🔒 Security Architecture

### Authentication Flow

```
Client                 Server
  │                      │
  │  POST /login        │
  │──────────────────────▶│
  │                      │
  │  Verify credentials  │
  │  ←──────────────────│
  │                      │
  │  Generate JWT      │
  │  ←──────────────────│
  │                      │
  │  Return token       │
  │──────────────────────▶│
  │                      │
  ▼                      ▼
Store token         (JWT valid for 7 days)
```

### Authorization

**Role-Based Access Control**:
```javascript
const checkPermission = (user, resource, action) => {
  const roles = {
    admin: ['read', 'write', 'delete', 'manage_users'],
    editor: ['read', 'write', 'delete'],
    viewer: ['read']
  };
  
  return roles[user.role]?.includes(action);
};
```

### Security Best Practices

1. **Password Hashing**: bcrypt with 10+ rounds
2. **JWT Secret**: Environment variable, never in code
3. **CORS**: Strict origin validation
4. **Rate Limiting**: Prevent brute force
5. **Input Validation**: Sanitize all inputs
6. **SQL Injection**: Parameterized queries only

---

## ⚡ Performance Optimizations

### Frontend Optimizations

**Virtualization**:
- react-window for large lists
- Renders only visible items
- O(n) → O(1) for rendering

**Code Splitting**:
- Lazy route loading
- Split by route
- Reduce initial bundle size

**Memoization**:
- React.memo for expensive components
- useMemo for calculations
- useCallback for event handlers

### Backend Optimizations

**Database Indexes**:
- Indexed columns for fast queries
- Composite indexes for complex queries
- Regular index maintenance

**Caching**:
- Redis for session caching (planned)
- Memory cache for frequently accessed notes
- CDN for static assets

---

## 🧪 Testing Architecture

### Test Structure

```
tests/
├── unit/
│   ├── components/
│   ├── hooks/
│   ├── utils/
│   └── api/
├── integration/
│   ├── auth/
│   ├── notes/
│   └── collaboration/
└── e2e/
    ├── login.spec.js
    ├── note-creation.spec.js
    └── collaboration.spec.js
```

### Testing Strategy

**Unit Tests**: Jest + React Testing Library
- Test components in isolation
- Mock API calls
- Test hooks and utils

**Integration Tests**: Supertest
- Test API endpoints
- Test database operations
- Test middleware

**E2E Tests**: Playwright
- Test user flows
- Test across browsers
- Test mobile/responsive

---

## 🚀 Deployment Architecture

### Development
```bash
npm run dev
```
- Vite dev server: localhost:5173
- Express server: localhost:3000
- WebSocket server: localhost:8080

### Production
```bash
npm run build
npm start
```
- Static files: dist/
- Single Node.js process
- PM2 for process management

### Docker Deployment
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]
```

---

## 📊 Monitoring & Logging

### Logging Strategy

**Frontend**:
```javascript
console.log('DEBUG:', data);
console.warn('WARN:', message);
console.error('ERROR:', error);
```

**Backend**:
```javascript
const logger = require('./logging-module');
logger.info('Request:', { method, path, userId });
logger.error('Error:', { error, stack });
```

**Log Levels**:
- DEBUG: Detailed debugging info
- INFO: General information
- WARN: Warning messages
- ERROR: Error conditions
- FATAL: Critical failures

---

## 🔗 System Diagrams

### Data Flow: User Creates Note

```
Browser            Vite          Express          SQLite
  │                 │                │
  │  Create note    │                │
  │────────────────▶│                │
  │                 │  INSERT        │
  │                 │─────────────────▶│
  │                 │                │
  │  Success       │                │
  │◀────────────────│                │
  │                 │                │
  │  Broadcast      │                │
  │────────────────▶│                │
  │                 │                │
  ▼                 │                ▼
Update UI         WebSocket       Store
```

### Data Flow: AI Query

```
Browser            Express         Ollama          SQLite
  │                 │                │
  │  Query         │                │
  │────────────────▶│                │
  │                 │  Search        │
  │                 │─────────────────▶│
  │                 │                │
  │  Results       │                │
  │◀────────────────│                │
  │                 │                │
  │  Generate      │                │
  │────────────────▶│                │
  │                 │  Prompt       │
  │                 │─────────────────▶│
  │                 │                │
  │  Response      │                │
  │◀────────────────│                │
  │                 │                │
  ▼                 ▼                ▼
Display          Server           Database
```

---

## 🎨 Component Architecture Patterns

### Presentational Components

```javascript
// No state, pure UI
const NoteCard = ({ note, onClick, onDelete }) => {
  return (
    <div className="note-card" onClick={() => onClick(note.id)}>
      <h3>{note.title}</h3>
      <p>{note.content}</p>
      <button onClick={(e) => {
        e.stopPropagation();
        onDelete(note.id);
      }}>Delete</button>
    </div>
  );
};
```

### Container Components

```javascript
// Manages state
const NotesView = () => {
  const { notes, loading, error } = useNotes();
  
  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;
  
  return (
    <div className="notes-view">
      {notes.map(note => <NoteCard key={note.id} note={note} />)}
    </div>
  );
};
```

### Custom Hooks

```javascript
const useLocalStorage = (key, initialValue) => {
  const [value, setValue] = useState(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });
  
  const setStoredValue = (newValue) => {
    setValue(newValue);
    localStorage.setItem(key, JSON.stringify(newValue));
  };
  
  return [value, setStoredValue];
};
```

---

## 📚 Resources

- **Setup Guide**: SETUP.md
- **API Reference**: ../API_REFERENCE.md
- **Testing Guide**: ../TESTING.md
- **Deployment Guide**: DEPLOYMENT.md
- **Database Schema**: ../DATABASE_SCHEMA.md

---

**Last Updated**: January 23, 2026  
**GLASSYDASH Version**: 0.67 (Beta)