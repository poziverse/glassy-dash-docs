# Testing Guide

Comprehensive guide to testing GLASSYDASH, including unit tests, integration tests, and E2E tests.

---

## 🧪 Testing Overview

GLASSYDASH uses a **multi-layered testing strategy**:

- **Unit Tests**: Test individual functions and components
- **Integration Tests**: Test API endpoints and database operations
- **E2E Tests**: Test complete user flows
- **Visual Regression Tests**: Ensure UI consistency

**Testing Stack**:
- **Jest**: Unit and integration tests
- **React Testing Library**: Component testing
- **Playwright**: E2E testing
- **Supertest**: API testing

---

## 📦 Installation

### Install Testing Dependencies

```bash
npm install --save-dev \
  @testing-library/react \
  @testing-library/jest-dom \
  @testing-library/user-event \
  @playwright/test \
  supertest \
  jest \
  jest-environment-jsdom \
  ts-jest
```

### Setup Playwright

```bash
npx playwright install
npx playwright install-deps
```

---

## 🧩 Unit Testing

### Test Structure

```
tests/unit/
├── components/
│   ├── NoteCard.test.jsx
│   ├── Composer.test.jsx
│   └── SearchBar.test.jsx
├── hooks/
│   ├── useNotes.test.js
│   └── useLocalStorage.test.js
├── utils/
│   ├── formatDate.test.js
│   └── sanitizeHTML.test.js
└── contexts/
    ├── AuthContext.test.js
    └── NotesContext.test.js
```

### Component Testing

**Example: NoteCard Component**

```jsx
// tests/unit/components/NoteCard.test.jsx
import { render, screen, fireEvent } from '@testing-library/react';
import { NoteCard } from '../../../src/components/NoteCard';

describe('NoteCard', () => {
  const mockNote = {
    id: 1,
    title: 'Test Note',
    content: 'Test content',
    tags: ['work', 'important'],
    pinned: false,
    created_at: '2026-01-23T12:00:00Z',
    updated_at: '2026-01-23T12:00:00Z'
  };

  it('renders note title', () => {
    render(<NoteCard note={mockNote} />);
    expect(screen.getByText('Test Note')).toBeInTheDocument();
  });

  it('renders note content', () => {
    render(<NoteCard note={mockNote} />);
    expect(screen.getByText('Test content')).toBeInTheDocument();
  });

  it('calls onClick when clicked', () => {
    const handleClick = jest.fn();
    render(<NoteCard note={mockNote} onClick={handleClick} />);
    
    fireEvent.click(screen.getByText('Test Note'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('displays tags', () => {
    render(<NoteCard note={mockNote} />);
    expect(screen.getByText('work')).toBeInTheDocument();
    expect(screen.getByText('important')).toBeInTheDocument();
  });

  it('shows pinned indicator when pinned', () => {
    const pinnedNote = { ...mockNote, pinned: true };
    render(<NoteCard note={pinnedNote} />);
    expect(screen.getByLabelText('pinned')).toBeInTheDocument();
  });
});
```

**Best Practices**:
- Test user behavior, not implementation details
- Use `screen` queries instead of `container` queries
- Test accessibility with `getByRole` and `getByLabelText`
- Mock external dependencies
- Keep tests focused and small

### Hook Testing

**Example: useLocalStorage Hook**

```javascript
// tests/unit/hooks/useLocalStorage.test.js
import { renderHook, act } from '@testing-library/react';
import { useLocalStorage } from '../../../src/hooks/useLocalStorage';

describe('useLocalStorage', () => {
  beforeEach(() => {
    localStorage.clear();
  });

  it('initializes with initial value', () => {
    const { result } = renderHook(() => 
      useLocalStorage('test-key', 'initial')
    );
    
    expect(result.current[0]).toBe('initial');
  });

  it('stores value in localStorage', () => {
    const { result } = renderHook(() => 
      useLocalStorage('test-key', 'initial')
    );
    
    act(() => {
      result.current[1]('updated');
    });
    
    expect(result.current[0]).toBe('updated');
    expect(localStorage.getItem('test-key')).toBe('"updated"');
  });

  it('loads value from localStorage', () => {
    localStorage.setItem('test-key', '"stored"');
    
    const { result } = renderHook(() => 
      useLocalStorage('test-key', 'initial')
    );
    
    expect(result.current[0]).toBe('stored');
  });

  it('handles JSON objects', () => {
    const object = { name: 'test', count: 1 };
    const { result } = renderHook(() => 
      useLocalStorage('test-key', {})
    );
    
    act(() => {
      result.current[1](object);
    });
    
    expect(result.current[0]).toEqual(object);
  });
});
```

### Utility Function Testing

**Example: formatDate Function**

```javascript
// tests/unit/utils/formatDate.test.js
import { formatDate } from '../../../src/utils/formatDate';

describe('formatDate', () => {
  it('formats ISO date string', () => {
    const date = '2026-01-23T12:00:00Z';
    expect(formatDate(date)).toBe('Jan 23, 2026');
  });

  it('handles different date formats', () => {
    expect(formatDate('2026-01-23')).toBe('Jan 23, 2026');
  expect(formatDate(new Date('2026-01-23'))).toBe('Jan 23, 2026');
  });

  it('returns empty string for invalid date', () => {
    expect(formatDate('invalid')).toBe('');
  });

  it('formats relative time for recent dates', () => {
    const now = new Date();
    const oneHourAgo = new Date(now - 60 * 60 * 1000);
    expect(formatDate(oneHourAgo.toISOString())).toBe('1 hour ago');
  });
});
```

---

## 🔗 Integration Testing

### API Endpoint Testing

**Example: Notes API**

```javascript
// tests/integration/notes.test.js
const request = require('supertest');
const { app } = require('../../server/index');
const db = require('../../server/db');

describe('Notes API', () => {
  let token;
  let userId;

  beforeAll(async () => {
    // Setup test database
    await db.run('INSERT INTO users (username, password, role) VALUES (?, ?, ?)', 
      ['testuser', '$2b$10$hash', 'user']
    );
    
    // Login to get token
    const response = await request(app)
      .post('/api/auth/login')
      .send({ username: 'testuser', password: 'password' });
    
    token = response.body.token;
    userId = response.body.user.id;
  });

  afterAll(async () => {
    // Cleanup
    await db.run('DELETE FROM notes WHERE user_id = ?', [userId]);
    await db.run('DELETE FROM users WHERE username = ?', ['testuser']);
  });

  describe('POST /api/notes', () => {
    it('creates a new note', async () => {
      const noteData = {
        title: 'Test Note',
        content: 'Test content',
        type: 'rich'
      };

      const response = await request(app)
        .post('/api/notes')
        .set('Authorization', `Bearer ${token}`)
        .send(noteData)
        .expect(201);

      expect(response.body).toHaveProperty('id');
      expect(response.body.title).toBe(noteData.title);
      expect(response.body.content).toBe(noteData.content);
    });

    it('returns 400 for missing title', async () => {
      const response = await request(app)
        .post('/api/notes')
        .set('Authorization', `Bearer ${token}`)
        .send({ content: 'Test' })
        .expect(400);

      expect(response.body.error).toBe('Title is required');
    });

    it('returns 401 without authentication', async () => {
      const response = await request(app)
        .post('/api/notes')
        .send({ title: 'Test', content: 'Test' })
        .expect(401);

      expect(response.body.error).toBe('No token provided');
    });
  });

  describe('GET /api/notes', () => {
    it('returns all notes for user', async () => {
      const response = await request(app)
        .get('/api/notes')
        .set('Authorization', `Bearer ${token}`)
        .expect(200);

      expect(Array.isArray(response.body.notes)).toBe(true);
      expect(response.body.notes.length).toBeGreaterThan(0);
    });

    it('supports pagination', async () => {
      const response = await request(app)
        .get('/api/notes?page=1&limit=10')
        .set('Authorization', `Bearer ${token}`)
        .expect(200);

      expect(response.body).toHaveProperty('pagination');
      expect(response.body.pagination.page).toBe(1);
      expect(response.body.pagination.limit).toBe(10);
    });
  });
});
```

### Database Testing

```javascript
// tests/integration/database.test.js
const db = require('../../server/db');

describe('Database Operations', () => {
  let testUserId;

  beforeEach(async () => {
    const result = await db.run(
      'INSERT INTO users (username, password, role) VALUES (?, ?, ?)',
      ['testuser', '$2b$10$hash', 'user']
    );
    testUserId = result.lastID;
  });

  afterEach(async () => {
    await db.run('DELETE FROM users WHERE id = ?', [testUserId]);
  });

  it('inserts user successfully', async () => {
    const user = await db.get('SELECT * FROM users WHERE id = ?', [testUserId]);
    expect(user).toBeDefined();
    expect(user.username).toBe('testuser');
    expect(user.role).toBe('user');
  });

  it('updates user successfully', async () => {
    await db.run(
      'UPDATE users SET role = ? WHERE id = ?',
      ['admin', testUserId]
    );
    
    const user = await db.get('SELECT * FROM users WHERE id = ?', [testUserId]);
    expect(user.role).toBe('admin');
  });

  it('deletes user successfully', async () => {
    await db.run('DELETE FROM users WHERE id = ?', [testUserId]);
    
    const user = await db.get('SELECT * FROM users WHERE id = ?', [testUserId]);
    expect(user).toBeUndefined();
  });
});
```

---

## 🎭 End-to-End Testing

### E2E Test Structure

```
tests/e2e/
├── auth.spec.js
├── note-creation.spec.js
├── note-editing.spec.js
├── search.spec.js
├── collaboration.spec.js
└── ai-search.spec.js
```

### Basic E2E Test

**Example: User Login**

```javascript
// tests/e2e/auth.spec.js
import { test, expect } from '@playwright/test';

test.describe('Authentication', () => {
  test('user can log in with valid credentials', async ({ page }) => {
    await page.goto('http://localhost:5173');
    
    // Click login button
    await page.click('button:has-text("Login")');
    
    // Fill in credentials
    await page.fill('input[name="username"]', 'admin');
    await page.fill('input[name="password"]', 'admin');
    
    // Submit form
    await page.click('button[type="submit"]');
    
    // Wait for redirect and check URL
    await page.waitForURL('**/dashboard');
    expect(page.url()).toContain('/dashboard');
  });

  test('shows error with invalid credentials', async ({ page }) => {
    await page.goto('http://localhost:5173');
    
    await page.click('button:has-text("Login")');
    await page.fill('input[name="username"]', 'invalid');
    await page.fill('input[name="password"]', 'invalid');
    await page.click('button[type="submit"]');
    
    // Check for error message
    const errorMessage = await page.locator('.error-message').textContent();
    expect(errorMessage).toContain('Invalid credentials');
  });
});
```

### Note Creation Test

```javascript
// tests/e2e/note-creation.spec.js
import { test, expect } from '@playwright/test';

test.describe('Note Creation', () => {
  test.beforeEach(async ({ page }) => {
    // Login before each test
    await page.goto('http://localhost:5173');
    await page.fill('input[name="username"]', 'admin');
    await page.fill('input[name="password"]', 'admin');
    await page.click('button[type="submit"]');
    await page.waitForURL('**/dashboard');
  });

  test('creates a new note', async ({ page }) => {
    // Click new note button
    await page.click('button:has-text("New Note")');
    
    // Wait for composer to open
    await page.waitForSelector('.composer');
    
    // Fill in note details
    await page.fill('input[name="title"]', 'Test Note');
    await page.fill('textarea[name="content"]', 'Test content here');
    
    // Save note
    await page.click('button:has-text("Save")');
    
    // Wait for note to appear in list
    await page.waitForSelector('.note-card:has-text("Test Note")');
    
    // Verify note was created
    const noteTitle = await page.locator('.note-card').first().textContent();
    expect(noteTitle).toContain('Test Note');
  });

  test('adds tags to note', async ({ page }) => {
    await page.click('button:has-text("New Note")');
    await page.waitForSelector('.composer');
    
    await page.fill('input[name="title"]', 'Tagged Note');
    await page.fill('textarea[name="content"]', 'Content');
    
    // Add tags
    await page.fill('input[placeholder="Add tag"]', 'work');
    await page.press('input[placeholder="Add tag"]', 'Enter');
    
    await page.fill('input[placeholder="Add tag"]', 'important');
    await page.press('input[placeholder="Add tag"]', 'Enter');
    
    await page.click('button:has-text("Save")');
    
    // Verify tags are displayed
    await page.waitForSelector('.note-card .tag:has-text("work")');
    await page.waitForSelector('.note-card .tag:has-text("important")');
  });

  test('pins a note', async ({ page }) => {
    await page.click('button:has-text("New Note")');
    await page.waitForSelector('.composer');
    
    await page.fill('input[name="title"]', 'Pinned Note');
    await page.click('button[aria-label="Pin note"]');
    
    await page.click('button:has-text("Save")');
    
    // Verify pinned indicator
    const pinnedIcon = await page.locator('.note-card .pinned-icon').count();
    expect(pinnedIcon).toBeGreaterThan(0);
  });
});
```

### Search Test

```javascript
// tests/e2e/search.spec.js
import { test, expect } from '@playwright/test';

test.describe('Search', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('http://localhost:5173');
    await page.fill('input[name="username"]', 'admin');
    await page.fill('input[name="password"]', 'admin');
    await page.click('button[type="submit"]');
    await page.waitForURL('**/dashboard');
  });

  test('searches for notes by title', async ({ page }) => {
    // Type in search bar
    await page.fill('input[placeholder="Search notes..."]', 'test');
    
    // Wait for search results
    await page.waitForTimeout(500);
    
    // Verify only matching notes appear
    const notes = await page.locator('.note-card').count();
    expect(notes).toBeGreaterThan(0);
    
    const titles = await page.locator('.note-card h3').allTextContents();
    titles.forEach(title => {
      expect(title.toLowerCase()).toContain('test');
    });
  });

  test('filters by tags', async ({ page }) => {
    // Click tag filter
    await page.click('.tag:has-text("work")');
    
    // Wait for filtering
    await page.waitForTimeout(500);
    
    // Verify all notes have work tag
    const notes = await page.locator('.note-card');
    const count = await notes.count();
    
    for (let i = 0; i < count; i++) {
      const hasWorkTag = await notes.nth(i)
        .locator('.tag:has-text("work")')
        .count();
      expect(hasWorkTag).toBeGreaterThan(0);
    }
  });
});
```

---

## 🎨 Visual Regression Testing

### Setup Playwright for Visual Testing

```javascript
// playwright.config.js
module.exports = {
  use: {
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    trace: 'on-first-retry'
  }
};
```

### Visual Test Example

```javascript
// tests/e2e/visual.spec.js
import { test, expect } from '@playwright/test';

test.describe('Visual Regression', () => {
  test('dashboard matches screenshot', async ({ page }) => {
    await page.goto('http://localhost:5173');
    await page.fill('input[name="username"]', 'admin');
    await page.fill('input[name="password"]', 'admin');
    await page.click('button[type="submit"]');
    await page.waitForURL('**/dashboard');
    
    // Take screenshot and compare
    await expect(page).toHaveScreenshot('dashboard.png');
  });

  test('note card matches screenshot', async ({ page }) => {
    await page.goto('http://localhost:5173');
    // Login and create note...
    
    const noteCard = page.locator('.note-card').first();
    await expect(noteCard).toHaveScreenshot('note-card.png');
  });
});
```

---

## 🚀 Running Tests

### Run All Tests

```bash
npm test
```

### Run Unit Tests

```bash
npm run test:unit
```

### Run Integration Tests

```bash
npm run test:integration
```

### Run E2E Tests

```bash
npm run test:e2e
```

### Watch Mode

```bash
npm run test:watch
```

### Coverage Report

```bash
npm run test:coverage
```

### Run Specific Test File

```bash
npm test NoteCard.test.jsx
```

### Run Tests Matching Pattern

```bash
npm test -- --testNamePattern="NoteCard"
```

---

## 📊 Coverage

### Coverage Configuration

```javascript
// jest.config.js
module.exports = {
  collectCoverageFrom: [
    'src/**/*.{js,jsx}',
    '!src/**/*.d.ts',
    '!src/**/*.test.{js,jsx}',
    '!src/**/__tests__/**'
  ],
  coverageThresholds: {
    global: {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70
    }
  }
};
```

### Generate Coverage Report

```bash
npm run test:coverage
```

View report at `coverage/lcov-report/index.html`

---

## 🎯 Testing Best Practices

### General Guidelines

1. **Write Tests Before Code** (TDD)
   - Write failing test first
   - Implement minimal code to pass
   - Refactor

2. **Keep Tests Independent**
   - No test should depend on another
   - Each test should be self-contained

3. **Use Descriptive Names**
   - `it('creates a new note')` ✅
   - `it('test1')` ❌

4. **Test Behavior, Not Implementation**
   - Test what user sees
   - Don't test internal state

5. **Mock External Dependencies**
   - API calls
   - Database
   - Third-party services

### Performance

1. **Run Tests in Parallel**
   - Jest runs tests in parallel by default
   - Playwright tests can run in parallel browsers

2. **Use BeforeEach/AfterEach**
   - Setup/teardown for each test
   - Keep tests isolated

3. **Avoid Sleep/Wait**
   - Use explicit waits
   - `waitForSelector` instead of `waitForTimeout`

### Maintenance

1. **Review Failing Tests**
   - Fix immediately
   - Don't ignore failing tests

2. **Refactor Tests**
   - Keep DRY (Don't Repeat Yourself)
   - Extract common logic to helpers

3. **Update Tests with Code**
   - Tests should always pass
   - Update when changing features

---

## 🔍 Debugging Tests

### Debug Unit Tests

```javascript
// Use test.only to run single test
it.only('creates a new note', () => {
  // Test code
});

// Use debugger statement
it('debug test', () => {
  const value = someFunction();
  debugger; // Execution stops here
  expect(value).toBe('expected');
});
```

### Debug E2E Tests

```javascript
// tests/e2e/debug.spec.js
import { test, expect } from '@playwright/test';

test('debug test', async ({ page, context }) => {
  // Slow down actions
  await context.setDefaultTimeout(60000);
  
  // Pause test execution
  await page.pause();
  
  // Continue with test
  await page.goto('http://localhost:5173');
});
```

### View Test Output

```bash
# Verbose output
npm test -- --verbose

# Silent output
npm test -- --silent

# Show only failed tests
npm test -- --onlyFailures
```

---

## 📚 Resources

- **Setup Guide**: SETUP.md
- **Architecture Guide**: ARCHITECTURE.md
- **API Guide**: API.md
- **Deployment Guide**: DEPLOYMENT.md

---

**Last Updated**: January 23, 2026  
**GLASSYDASH Version**: 0.67 (Beta)