# API Development Guide

Complete guide to working with GLASSYDASH's REST API and WebSocket endpoints.

---

## 📡 API Overview

GLASSYDASH exposes a **RESTful API** for all CRUD operations and a **WebSocket API** for real-time updates.

**Base URL**: `http://localhost:3000/api`

**Authentication**: JWT Bearer Token required for most endpoints

**Response Format**: JSON

---

## 🔐 Authentication

### Login

**Endpoint**: `POST /api/auth/login`

**Request Body**:
```json
{
  "username": "string",
  "password": "string"
}
```

**Success Response** (200):
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 1,
    "username": "admin",
    "email": "admin@example.com",
    "role": "admin"
  }
}
```

**Error Response** (401):
```json
{
  "error": "Invalid credentials"
}
```

**Example**:
```bash
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin"}'
```

### Register

**Endpoint**: `POST /api/auth/register`

**Request Body**:
```json
{
  "username": "string",
  "email": "string",
  "password": "string"
}
```

**Success Response** (201):
```json
{
  "user": {
    "id": 2,
    "username": "newuser",
    "email": "newuser@example.com",
    "role": "user"
  }
}
```

**Error Response** (400):
```json
{
  "error": "Username already exists"
}
```

### Get Current User

**Endpoint**: `GET /api/auth/me`

**Headers**: `Authorization: Bearer <token>`

**Success Response** (200):
```json
{
  "id": 1,
  "username": "admin",
  "email": "admin@example.com",
  "role": "admin",
  "created_at": "2026-01-23T12:00:00Z"
}
```

**Example**:
```bash
curl -X GET http://localhost:3000/api/auth/me \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

### Logout

**Endpoint**: `POST /api/auth/logout`

**Headers**: `Authorization: Bearer <token>`

**Success Response** (200):
```json
{
  "message": "Logged out successfully"
}
```

---

## 📝 Notes API

### List Notes

**Endpoint**: `GET /api/notes`

**Headers**: `Authorization: Bearer <token>`

**Query Parameters**:
- `page`: Page number (default: 1)
- `limit`: Items per page (default: 20)
- `sort`: Sort field (created_at, updated_at, title)
- `order`: Sort order (asc, desc)

**Success Response** (200):
```json
{
  "notes": [
    {
      "id": 1,
      "title": "My Note",
      "content": "Note content",
      "type": "rich",
      "tags": ["work", "important"],
      "pinned": true,
      "created_at": "2026-01-23T12:00:00Z",
      "updated_at": "2026-01-23T12:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 50,
    "totalPages": 3
  }
}
```

**Example**:
```bash
curl -X GET "http://localhost:3000/api/notes?page=1&limit=20&sort=created_at&order=desc" \
  -H "Authorization: Bearer <token>"
```

### Get Single Note

**Endpoint**: `GET /api/notes/:id`

**Headers**: `Authorization: Bearer <token>`

**Success Response** (200):
```json
{
  "id": 1,
  "title": "My Note",
  "content": "Full note content here...",
  "type": "rich",
  "tags": ["work", "important"],
  "pinned": true,
  "user_id": 1,
  "created_at": "2026-01-23T12:00:00Z",
  "updated_at": "2026-01-23T12:00:00Z"
}
```

**Error Response** (404):
```json
{
  "error": "Note not found"
}
```

### Create Note

**Endpoint**: `POST /api/notes`

**Headers**: `Authorization: Bearer <token>`

**Request Body**:
```json
{
  "title": "string",
  "content": "string",
  "type": "rich|plain|checklist",
  "tags": ["string"],
  "pinned": boolean
}
```

**Success Response** (201):
```json
{
  "id": 2,
  "title": "New Note",
  "content": "Note content",
  "type": "rich",
  "tags": ["work"],
  "pinned": false,
  "user_id": 1,
  "created_at": "2026-01-23T12:30:00Z",
  "updated_at": "2026-01-23T12:30:00Z"
}
```

**Error Response** (400):
```json
{
  "error": "Title is required"
}
```

**Example**:
```bash
curl -X POST http://localhost:3000/api/notes \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "New Note",
    "content": "Note content here",
    "type": "rich",
    "tags": ["work"],
    "pinned": false
  }'
```

### Update Note

**Endpoint**: `PUT /api/notes/:id`

**Headers**: `Authorization: Bearer <token>`

**Request Body**: (all fields optional)
```json
{
  "title": "string",
  "content": "string",
  "tags": ["string"],
  "pinned": boolean
}
```

**Success Response** (200):
```json
{
  "id": 1,
  "title": "Updated Title",
  "content": "Updated content",
  "type": "rich",
  "tags": ["work", "updated"],
  "pinned": true,
  "created_at": "2026-01-23T12:00:00Z",
  "updated_at": "2026-01-23T13:00:00Z"
}
```

**Error Response** (403):
```json
{
  "error": "Permission denied"
}
```

**Error Response** (404):
```json
{
  "error": "Note not found"
}
```

### Delete Note

**Endpoint**: `DELETE /api/notes/:id`

**Headers**: `Authorization: Bearer <token>`

**Success Response** (200):
```json
{
  "message": "Note deleted successfully"
}
```

**Error Response** (403):
```json
{
  "error": "Permission denied"
}
```

**Error Response** (404):
```json
{
  "error": "Note not found"
}
```

### Search Notes

**Endpoint**: `POST /api/notes/search`

**Headers**: `Authorization: Bearer <token>`

**Request Body**:
```json
{
  "query": "string",
  "tags": ["string"],
  "date_from": "ISO8601",
  "date_to": "ISO8601"
}
```

**Success Response** (200):
```json
{
  "notes": [
    {
      "id": 1,
      "title": "Matching Note",
      "content": "Content with search term",
      "type": "rich",
      "tags": ["work"],
      "pinned": false,
      "created_at": "2026-01-23T12:00:00Z",
      "updated_at": "2026-01-23T12:00:00Z"
    }
  ],
  "total": 1
}
```

**Example**:
```bash
curl -X POST http://localhost:3000/api/notes/search \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "search term",
    "tags": ["work"],
    "date_from": "2026-01-01T00:00:00Z",
    "date_to": "2026-12-31T23:59:59Z"
  }'
```

---

## 👥 Users API

### List Users

**Endpoint**: `GET /api/users`

**Headers**: `Authorization: Bearer <token>`

**Required Role**: admin

**Success Response** (200):
```json
{
  "users": [
    {
      "id": 1,
      "username": "admin",
      "email": "admin@example.com",
      "role": "admin",
      "created_at": "2026-01-23T12:00:00Z"
    },
    {
      "id": 2,
      "username": "user1",
      "email": "user1@example.com",
      "role": "user",
      "created_at": "2026-01-23T13:00:00Z"
    }
  ]
}
```

**Error Response** (403):
```json
{
  "error": "Admin access required"
}
```

### Get User

**Endpoint**: `GET /api/users/:id`

**Headers**: `Authorization: Bearer <token>`

**Success Response** (200):
```json
{
  "id": 1,
  "username": "admin",
  "email": "admin@example.com",
  "role": "admin",
  "created_at": "2026-01-23T12:00:00Z",
  "updated_at": "2026-01-23T12:00:00Z"
}
```

### Update User

**Endpoint**: `PUT /api/users/:id`

**Headers**: `Authorization: Bearer <token>`

**Request Body**: (all fields optional)
```json
{
  "email": "string",
  "password": "string",
  "role": "admin|editor|viewer"
}
```

**Success Response** (200):
```json
{
  "id": 2,
  "username": "user1",
  "email": "updated@example.com",
  "role": "editor",
  "created_at": "2026-01-23T13:00:00Z",
  "updated_at": "2026-01-23T14:00:00Z"
}
```

**Error Response** (403):
```json
{
  "error": "Permission denied"
}
```

### Delete User

**Endpoint**: `DELETE /api/users/:id`

**Headers**: `Authorization: Bearer <token>`

**Required Role**: admin

**Success Response** (200):
```json
{
  "message": "User deleted successfully"
}
```

**Error Response** (403):
```json
{
  "error": "Cannot delete admin user"
}
```

---

## 🤝 Collaboration API

### Join Collaboration

**Endpoint**: `POST /api/collaboration/:noteId/join`

**Headers**: `Authorization: Bearer <token>`

**Success Response** (200):
```json
{
  "message": "Joined collaboration",
  "note_id": 1,
  "user_id": 2,
  "permission": "read"
}
```

### Leave Collaboration

**Endpoint**: `POST /api/collaboration/:noteId/leave`

**Headers**: `Authorization: Bearer <token>`

**Success Response** (200):
```json
{
  "message": "Left collaboration"
}
```

### Set Permission

**Endpoint**: `PUT /api/collaboration/:noteId/:userId`

**Headers**: `Authorization: Bearer <token>`

**Request Body**:
```json
{
  "permission": "read|write"
}
```

**Success Response** (200):
```json
{
  "message": "Permission updated",
  "permission": "write"
}
```

---

## 🤖 AI Search API

### AI Query

**Endpoint**: `POST /api/ai/search`

**Headers**: `Authorization: Bearer <token>`

**Request Body**:
```json
{
  "query": "string",
  "max_results": number
}
```

**Success Response** (200):
```json
{
  "answer": "AI-generated answer based on your notes",
  "sources": [
    {
      "id": 1,
      "title": "Relevant Note",
      "content": "Relevant content excerpt",
      "relevance": 0.95
    }
  ],
  "query": "search query"
}
```

**Error Response** (500):
```json
{
  "error": "AI service unavailable"
}
```

**Example**:
```bash
curl -X POST http://localhost:3000/api/ai/search \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What are my project deadlines?",
    "max_results": 5
  }'
```

---

## 🔌 WebSocket API

### Connection

**Endpoint**: `ws://localhost:8080`

**Connection Query Parameters**:
- `token`: JWT token for authentication

**Example**:
```javascript
const ws = new WebSocket('ws://localhost:8080?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...');
```

### Message Format

**Client → Server**:
```json
{
  "type": "note_updated|note_created|note_deleted|cursor_move",
  "data": {
    "note_id": number,
    "content": "string",
    "cursor_position": number
  }
}
```

**Server → Client**:
```json
{
  "type": "note_updated|note_created|note_deleted|cursor_move|user_joined|user_left",
  "data": {
    "note_id": number,
    "user_id": number,
    "username": "string",
    "timestamp": "ISO8601",
    "content": "string",
    "cursor_position": number
  }
}
```

### Event Types

**note_updated**: Broadcast when a note is edited
**note_created**: Broadcast when a note is created
**note_deleted**: Broadcast when a note is deleted
**cursor_move**: Broadcast when a collaborator moves cursor
**user_joined**: Broadcast when a user joins collaboration
**user_left**: Broadcast when a user leaves collaboration

### WebSocket Client Example

```javascript
class CollaborativeClient {
  constructor(token) {
    this.ws = new WebSocket(`ws://localhost:8080?token=${token}`);
    this.setupEventListeners();
  }

  setupEventListeners() {
    this.ws.onopen = () => {
      console.log('WebSocket connected');
    };

    this.ws.onmessage = (event) => {
      const message = JSON.parse(event.data);
      this.handleMessage(message);
    };

    this.ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };

    this.ws.onclose = () => {
      console.log('WebSocket disconnected');
      // Implement reconnection logic
    };
  }

  handleMessage(message) {
    switch (message.type) {
      case 'note_updated':
        this.updateNote(message.data);
        break;
      case 'cursor_move':
        this.updateCursor(message.data);
        break;
      case 'user_joined':
        this.showUserJoined(message.data);
        break;
      // Handle other event types
    }
  }

  sendUpdate(type, data) {
    this.ws.send(JSON.stringify({ type, data }));
  }

  updateNote(data) {
    // Update note in UI
    console.log('Note updated:', data);
  }

  updateCursor(data) {
    // Update cursor position in UI
    console.log('Cursor moved:', data);
  }

  close() {
    this.ws.close();
  }
}

// Usage
const token = localStorage.getItem('token');
const client = new CollaborativeClient(token);
```

---

## 📊 Response Codes

### Success Codes

- `200 OK`: Request succeeded
- `201 Created`: Resource created
- `204 No Content`: Request succeeded, no content returned

### Client Error Codes

- `400 Bad Request`: Invalid request data
- `401 Unauthorized`: Missing or invalid authentication
- `403 Forbidden`: Insufficient permissions
- `404 Not Found`: Resource not found
- `409 Conflict`: Resource conflict (duplicate, etc.)
- `422 Unprocessable Entity`: Valid JSON but semantically incorrect
- `429 Too Many Requests`: Rate limit exceeded

### Server Error Codes

- `500 Internal Server Error`: Unexpected server error
- `502 Bad Gateway`: Invalid response from upstream server
- `503 Service Unavailable`: Server temporarily unavailable
- `504 Gateway Timeout`: Upstream server timeout

---

## 🔄 Error Handling

### Standard Error Response Format

```json
{
  "error": "Error message",
  "details": {
    "field": "Additional error details"
  }
}
```

### Error Handling Best Practices

**Frontend**:
```javascript
async function createNote(noteData) {
  try {
    const response = await fetch('/api/notes', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(noteData)
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || 'Failed to create note');
    }

    return await response.json();
  } catch (error) {
    console.error('Create note error:', error);
    // Show user-friendly error message
    showErrorToast(error.message);
  }
}
```

**Rate Limiting**:
```javascript
async function fetchWithRetry(url, options, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await fetch(url, options);
      
      if (response.status === 429) {
        const retryAfter = response.headers.get('Retry-After');
        await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
        continue;
      }
      
      return response;
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
}
```

---

## 🧪 API Testing

### Using cURL

**Login**:
```bash
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin"}'
```

**Create Note**:
```bash
TOKEN="your-jwt-token"
curl -X POST http://localhost:3000/api/notes \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"Test Note","content":"Test content"}'
```

### Using Postman

1. Import API collection (if available)
2. Set base URL: `http://localhost:3000`
3. Add authentication token to headers
4. Test each endpoint

### Using JavaScript

```javascript
class APIClient {
  constructor(baseURL) {
    this.baseURL = baseURL;
    this.token = localStorage.getItem('token');
  }

  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const headers = {
      'Content-Type': 'application/json',
      ...options.headers
    };

    if (this.token) {
      headers['Authorization'] = `Bearer ${this.token}`;
    }

    const response = await fetch(url, {
      ...options,
      headers
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || 'Request failed');
    }

    return await response.json();
  }

  // Convenience methods
  get(endpoint) {
    return this.request(endpoint, { method: 'GET' });
  }

  post(endpoint, data) {
    return this.request(endpoint, {
      method: 'POST',
      body: JSON.stringify(data)
    });
  }

  put(endpoint, data) {
    return this.request(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data)
    });
  }

  delete(endpoint) {
    return this.request(endpoint, { method: 'DELETE' });
  }
}

// Usage
const api = new APIClient('http://localhost:3000/api');
const notes = await api.get('/notes');
const newNote = await api.post('/notes', {
  title: 'Test Note',
  content: 'Test content'
});
```

---

## 📚 Resources

- **Architecture Guide**: ARCHITECTURE.md
- **Setup Guide**: SETUP.md
- **Testing Guide**: TESTING.md
- **API Reference**: ../API_REFERENCE.md

---

**Last Updated**: January 23, 2026  
**GLASSYDASH Version**: 0.67 (Beta)