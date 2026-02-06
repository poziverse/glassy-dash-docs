# Document Cards Color & PIN Features Implementation Plan

**Created:** January 31, 2026  
**Status:** 📋 Planning  
**Priority:** High  
**Estimated Effort:** 4-6 hours

---

## Executive Summary

This plan details the implementation of color customization features for Document Cards to achieve feature parity with Note Cards. Currently, Document Cards support PIN functionality but lack the color customization available in Note Cards. This implementation will enable users to assign colors to documents, providing visual organization and improved user experience.

---

## Current State Analysis

### Note Cards Features (Reference Implementation)

**Color Features:**
- 12 color options: default, red, yellow, green, blue, purple, peach, sage, mint, sky, sand, mauve
- Color stored in note object as `color` field
- Color picker in Composer modal using ColorDot component
- Background color applied using `bgFor()` utility from themes.js
- Color persists through localStorage/API
- Responsive color preview in card view

**PIN Features:**
- Toggle pin state via pin button on hover
- Pinned notes displayed in "Pinned" section
- Pin icon filled when pinned, outline when unpinned
- Pin state persists through localStorage/API
- Pin button disabled in multi-select mode

### Document Cards Current State

**Existing Features:**
- ✅ PIN functionality (togglePin in docsStore)
- ✅ Pin button in toolbar with icon
- ✅ Pinned documents visually distinguished (yellow icon background)
- ✅ Grid and list view modes
- ✅ Multi-select mode
- ✅ Bulk operations

**Missing Features:**
- ❌ Color customization
- ❌ Color picker UI
- ❌ Color field in document data structure
- ❌ Color persistence to API/database
- ❌ Color preview in card view

### Data Structure Comparison

**Note Object (Reference):**
```javascript
{
  id: string,
  color: string,           // ✅ Present
  pinned: boolean,         // ✅ Present
  // ... other fields
}
```

**Document Object (Current):**
```javascript
{
  id: string,
  title: string,
  content: string,
  folderId: string,
  tags: [],
  pinned: boolean,         // ✅ Present
  createdAt: string,
  updatedAt: string,
  deletedAt: null,
  // ❌ MISSING: color field
}
```

---

## Feature Gap Analysis

| Feature | Note Cards | Document Cards | Gap |
|---------|-----------|----------------|-----|
| Color Customization | ✅ 12 colors | ❌ None | **High** |
| Color Picker UI | ✅ Modal/Composer | ❌ None | **High** |
| Color Persistence | ✅ API/Local | ❌ None | **High** |
| PIN Toggle | ✅ Hover button | ✅ Hover button | None |
| PIN Persistence | ✅ API/Local | ✅ API/Local | None |
| Pinned Section | ✅ Separate section | ✅ Integrated | None |

**Conclusion:** Primary gap is color customization features. PIN functionality is already implemented but needs color integration for visual consistency.

---

## Implementation Plan

### Phase 1: Backend & Data Layer (1.5 hours)

#### 1.1 Database Schema Update

**Files to Modify:**
- `server/routes/documents.js` (API routes)
- `server/db/migrations.js` (if migration system exists)

**Tasks:**
- Add `color` column to documents table
- Set default value to `'default'`
- Update migration scripts
- Ensure backward compatibility (existing documents get 'default' color)

**SQL Migration:**
```sql
ALTER TABLE documents ADD COLUMN color VARCHAR(20) DEFAULT 'default';

-- Update existing documents
UPDATE documents SET color = 'default' WHERE color IS NULL;
```

**Validation:**
```javascript
// Add color validation in API routes
const validColors = ['default', 'red', 'yellow', 'green', 'blue', 'purple', 
                    'peach', 'sage', 'mint', 'sky', 'sand', 'mauve'];
if (color && !validColors.includes(color)) {
  throw new Error('Invalid color');
}
```

#### 1.2 API Endpoints Update

**Endpoints to Update:**

**POST /api/documents** - Create document with color
```javascript
app.post('/api/documents', async (req, res) => {
  const { title, content, folderId, color = 'default', tags = [] } = req.body;
  
  // Validate color
  if (!validColors.includes(color)) {
    return res.status(400).json({ error: 'Invalid color' });
  }
  
  const newDoc = {
    id: generateId(),
    title,
    content,
    folderId,
    color,  // ✅ Add color
    tags,
    pinned: false,
    createdAt: new Date().toISOString(),
    updatedAt: new Date().toISOString(),
    deletedAt: null
  };
  
  // Save and return
  res.json(newDoc);
});
```

**PUT /api/documents/:id** - Update document including color
```javascript
app.put('/api/documents/:id', async (req, res) => {
  const { id } = req.params;
  const updates = req.body;
  
  // Validate color if present
  if (updates.color && !validColors.includes(updates.color)) {
    return res.status(400).json({ error: 'Invalid color' });
  }
  
  // Update document
  const doc = await updateDocument(id, updates);
  res.json(doc);
});
```

**PATCH /api/documents/:id** - Partial updates
```javascript
app.patch('/api/documents/:id', async (req, res) => {
  const { id } = req.params;
  const updates = req.body;
  
  // Validate color if present
  if (updates.color && !validColors.includes(updates.color)) {
    return res.status(400).json({ error: 'Invalid color' });
  }
  
  const doc = await updateDocument(id, updates);
  res.json(doc);
});
```

**GET /api/documents** - Return documents with color
```javascript
app.get('/api/documents', async (req, res) => {
  const docs = await getAllDocuments();
  // Color is now included in response
  res.json(docs);
});
```

---

### Phase 2: Store & State Management (1 hour)

#### 2.1 Update docsStore

**File:** `glassy-dash/GLASSYDASH/src/stores/docsStore.js`

**Changes:**

**1. Add color field to newDoc creation:**
```javascript
createDoc: async (folderId = 'root') => {
  const newDoc = {
    id: crypto.randomUUID(),
    title: 'Untitled Document',
    content: '',
    folderId,
    color: 'default',  // ✅ Add default color
    tags: [],
    pinned: false,
    createdAt: new Date().toISOString(),
    updatedAt: new Date().toISOString(),
    deletedAt: null,
  }
  
  try {
    set({ loading: true, error: null })
    
    // Optimistic update
    set(state => ({
      docs: [newDoc, ...state.docs],
      activeDocId: state.activeDocId,
    }))
    
    // API call
    await apiRequest(API_BASE, {
      method: 'POST',
      body: JSON.stringify(newDoc),
    })
    
    // Reload to get server response
    await get().loadDocuments()
    
    set({ loading: false, error: null })
    return newDoc.id
  } catch (error) {
    // Rollback on error
    set(state => ({
      docs: state.docs.filter(d => d.id !== newDoc.id),
      loading: false,
      error: error.message,
    }))
    throw error
  }
}
```

**2. Add setColor function (similar to togglePin):**
```javascript
setColor: async (docId, color) => {
  const currentDoc = get().docs.find(d => d.id === docId)
  if (!currentDoc) return

  // Validate color
  const validColors = ['default', 'red', 'yellow', 'green', 'blue', 'purple', 
                     'peach', 'sage', 'mint', 'sky', 'sand', 'mauve']
  if (!validColors.includes(color)) {
    throw new Error('Invalid color')
  }

  try {
    set({ loading: true, error: null })

    // Optimistic update
    set(state => ({
      docs: state.docs.map(d =>
        d.id === docId ? { ...d, color, updatedAt: new Date().toISOString() } : d
      ),
    }))

    // API call
    await apiRequest(`${API_BASE}/${docId}`, {
      method: 'PATCH',
      body: JSON.stringify({ color }),
    })

    set({ loading: false, error: null })
  } catch (error) {
    // Rollback on error
    set(state => ({
      docs: state.docs.map(d => (d.id === docId ? currentDoc : d)),
      loading: false,
      error: error.message,
    }))
    throw error
  }
}
```

**3. Update store export to include setColor:**
```javascript
export const useDocsStore = create(
  persist(
    (set, get) => ({
      docs: [],
      trash: [],
      activeDocId: null,
      loading: false,
      error: null,

      // Existing methods...
      createDoc,
      updateDoc,
      deleteDoc,
      restoreDoc,
      permanentDeleteDoc,
      clearTrash,
      setActiveDoc,
      togglePin,
      addTag,
      removeTag,
      bulkDeleteDocs,
      bulkMoveDocs,
      clearError,

      // ✅ New method
      setColor,
    }),
    {
      name: 'glassy-docs-storage',
      partialize: state => ({
        activeDocId: state.activeDocId,
      }),
    }
  )
)
```

---

### Phase 3: UI Components (2 hours)

#### 3.1 Add Color Picker to Document Editor

**File:** `glassy-dash/GLASSYDASH/src/components/DocsView.jsx`

**Add imports:**
```javascript
import { ColorDot } from './ColorDot'
import { COLOR_ORDER, LIGHT_COLORS, DARK_COLORS } from '../themes'
import { Palette } from 'lucide-react'
import { Popover } from './Popover'
```

**Add state for color picker:**
```javascript
const [showColorPicker, setShowColorPicker] = useState(null) // docId or null
```

**Add color picker button in editor toolbar (after Pin button):**
```javascript
<div className="flex items-center gap-4 bg-black/20 p-2 border-b border-white/10 backdrop-blur-md">
  <button onClick={() => setActiveDoc(null)}>
    <ChevronRight className="rotate-180" size={20} />
  </button>
  
  <input
    type="text"
    value={activeDoc.title}
    onChange={e => updateDoc(activeDoc.id, { title: e.target.value })}
    placeholder="Untitled Document"
    className="bg-transparent border-none outline-none text-lg font-semibold text-gray-100 placeholder-gray-500 flex-1"
  />
  
  {/* ✅ Color Picker Button */}
  <div className="relative">
    <button
      onClick={() => setShowColorPicker(showColorPicker === 'editor' ? null : 'editor')}
      className="p-2 hover:bg-white/10 rounded-lg transition-colors"
      title="Document Color"
    >
      <Palette size={18} className="text-gray-400" />
    </button>
    
    {showColorPicker === 'editor' && (
      <Popover onClose={() => setShowColorPicker(null)}>
        <div className="grid grid-cols-6 gap-2 p-4">
          {COLOR_ORDER.filter(name => LIGHT_COLORS[name]).map(name => (
            <ColorDot
              key={name}
              name={name}
              selected={activeDoc?.color === name}
              onClick={() => {
                updateDoc(activeDoc.id, { color: name })
                setShowColorPicker(null)
              }}
              darkMode={dark}
            />
          ))}
        </div>
      </Popover>
    )}
  </div>
  
  <span className="text-xs text-gray-500">
    {activeDoc.updatedAt ? `Saved ${formatDate(activeDoc.updatedAt)}` : 'Unsaved'}
  </span>
  
  <button onClick={() => showDeleteConfirm(activeDoc.id)}>
    <Trash2 size={18} className="text-red-400" />
  </button>
</div>
```

#### 3.2 Add Color Picker to Document Cards

**Add color picker button in card footer (after Pin button):**
```javascript
<div
  className={`
    flex items-center text-xs text-gray-500
    ${viewMode === 'grid' ? 'mt-auto pt-3 border-t border-white/5 justify-between w-full' : 'ml-4 gap-4'}
  `}
>
  <span className="flex items-center gap-1">
    <Calendar size={12} />
    {formatDate(doc.updatedAt)}
  </span>

  {/* Actions */}
  <div className="flex items-center gap-1 opacity-0 group-hover:opacity-100 transition-opacity">
    {!showTrash && (
      <>
        {/* ✅ Color Picker Button for Cards */}
        <div className="relative">
          <button
            onClick={e => {
              e.stopPropagation()
              setShowColorPicker(showColorPicker === doc.id ? null : doc.id)
            }}
            className="p-1.5 rounded hover:bg-white/10 text-gray-400"
            title="Color"
          >
            <Palette size={14} />
          </button>
          
          {showColorPicker === doc.id && (
            <Popover onClose={() => setShowColorPicker(null)}>
              <div className="grid grid-cols-6 gap-2 p-4">
                {COLOR_ORDER.filter(name => LIGHT_COLORS[name]).map(name => (
                  <ColorDot
                    key={name}
                    name={name}
                    selected={doc.color === name}
                    onClick={e => {
                      e.stopPropagation()
                      updateDoc(doc.id, { color: name })
                      setShowColorPicker(null)
                    }}
                    darkMode={dark}
                  />
                ))}
              </div>
            </Popover>
          )}
        </div>
        
        {/* Existing Pin Button */}
        <button
          onClick={e => {
            e.stopPropagation()
            togglePin(doc.id)
          }}
          className={`p-1.5 rounded hover:bg-white/10 ${doc.pinned ? 'text-yellow-500' : 'text-gray-400'}`}
          title="Pin"
        >
          {doc.pinned ? (
            <Pin size={14} className="fill-current" />
          ) : (
            <Pin size={14} />
          )}
        </button>
      </>
    )}
    
    {/* Trash actions... */}
  </div>
</div>
```

#### 3.3 Apply Background Color to Document Cards

**Update card background style:**
```javascript
<div
  key={doc.id}
  data-testid="doc-card"
  data-doc-id={doc.id}
  onClick={() => {
    if (isSelectionMode) toggleSelection(doc.id)
    else if (!showTrash) setActiveDoc(doc.id)
  }}
  className={`
    group relative flex rounded-2xl border transition-all duration-300 cursor-pointer overflow-hidden
    ${viewMode === 'grid' ? 'flex-col h-[220px] p-5' : 'flex-row items-center p-4 h-auto'}
    ${
      selectedDocIds.includes(doc.id)
        ? 'bg-indigo-600/20 border-indigo-500'
        : 'border-white/10 hover:border-white/20 hover:shadow-xl'
    }
  `}
  style={{
    backgroundColor: doc.color && doc.color !== 'default' 
      ? bgFor(doc.color, dark, cardTransparency) 
      : 'rgba(255, 255, 255, 0.05)'
  }}
  onContextMenu={e => handleContextMenu(e, doc.id)}
>
  {/* Card content... */}
</div>
```

#### 3.4 Add Color to Document Editor Background

**Update editor glass card style:**
```javascript
<div
  className="max-w-4xl mx-auto glass-card rounded-2xl min-h-[800px] p-8 md:p-12 shadow-2xl border border-white/10 transition-all duration-300"
  style={{
    backgroundColor: activeDoc?.color && activeDoc.color !== 'default'
      ? bgFor(activeDoc.color, dark, cardTransparency)
      : bgFor('default', dark, cardTransparency),
    backdropFilter: 'blur(var(--glass-blur))',
    WebkitBackdropFilter: 'blur(var(--glass-blur))',
  }}
>
  <GlassyEditor
    content={activeDoc.content}
    onChange={content => updateDoc(activeDoc.id, { content })}
    data-testid="doc-editor"
  />
</div>
```

---

### Phase 4: Context Menu Integration (0.5 hour)

#### 4.1 Add Color Option to Context Menu

**File:** `glassy-dash/GLASSYDASH/src/components/DocsView.jsx`

**Add color picker to context menu:**
```javascript
{contextMenu && (
  <ContextMenu
    position={{ x: contextMenu.x, y: contextMenu.y }}
    onClose={() => setContextMenu(null)}
    dark={dark}
  >
    {showTrash ? (
      <>
        <ContextMenuItem
          icon={<RotateCcw size={16} />}
          label="Restore"
          onClick={() => {
            handleRestore(contextMenu.docId)
            setContextMenu(null)
          }}
        />
        <ContextMenuItem
          icon={<Trash2 size={16} />}
          label="Delete Permanently"
          danger
          onClick={() => {
            handlePermanentDelete(contextMenu.docId)
            setContextMenu(null)
          }}
        />
      </>
    ) : (
      <>
        <ContextMenuItem
          icon={<Edit size={16} />}
          label="Open / Rename"
          onClick={() => {
            const doc = docs.find(d => d.id === contextMenu.docId)
            if (doc) {
              setActiveDoc(doc.id)
            }
            setContextMenu(null)
          }}
        />
        
        {/* ✅ Add Color Option */}
        <div className="p-2">
          <div className="text-xs text-gray-400 mb-2 px-2">Color</div>
          <div className="grid grid-cols-6 gap-2">
            {COLOR_ORDER.filter(name => LIGHT_COLORS[name]).map(name => (
              <ColorDot
                key={name}
                name={name}
                selected={docs.find(d => d.id === contextMenu.docId)?.color === name}
                onClick={() => {
                  updateDoc(contextMenu.docId, { color: name })
                  setContextMenu(null)
                }}
                darkMode={dark}
              />
            ))}
          </div>
        </div>
        
        <ContextMenuSeparator dark={dark} />
        
        <ContextMenuItem
          icon={
            <Pin
              size={16}
              className={
                docs.find(d => d.id === contextMenu.docId)?.pinned ? 'fill-current' : ''
              }
            />
          }
          label={docs.find(d => d.id === contextMenu.docId)?.pinned ? 'Unpin' : 'Pin'}
          onClick={() => {
            togglePin(contextMenu.docId)
            setContextMenu(null)
          }}
        />
        <ContextMenuSeparator dark={dark} />
        <ContextMenuItem
          icon={<Trash2 size={16} />}
          label="Move to Trash"
          danger
          onClick={() => {
            showDeleteConfirm(contextMenu.docId)
            setContextMenu(null)
          }}
        />
      </>
    )}
  </ContextMenu>
)}
```

---

### Phase 5: Testing (1 hour)

#### 5.1 Unit Tests

**File:** `glassy-dash/GLASSYDASH/tests/docsStore.test.jsx`

```javascript
describe('docsStore - Color Features', () => {
  it('should create document with default color', async () => {
    const docId = await createDoc('root')
    const doc = docs.find(d => d.id === docId)
    expect(doc.color).toBe('default')
  })

  it('should update document color', async () => {
    const docId = await createDoc('root')
    await setColor(docId, 'red')
    const doc = docs.find(d => d.id === docId)
    expect(doc.color).toBe('red')
  })

  it('should reject invalid colors', async () => {
    const docId = await createDoc('root')
    await expect(setColor(docId, 'invalid')).rejects.toThrow('Invalid color')
  })

  it('should roll back color change on API error', async () => {
    const docId = await createDoc('root')
    const originalColor = docs.find(d => d.id === docId).color
    
    // Mock API failure
    jest.spyOn(global, 'fetch').mockRejectedValueOnce(new Error('API error'))
    
    await expect(setColor(docId, 'red')).rejects.toThrow()
    
    const doc = docs.find(d => d.id === docId)
    expect(doc.color).toBe(originalColor)
  })
})
```

#### 5.2 Integration Tests

**File:** `glassy-dash/GLASSYDASH/tests/e2e/documents-color.spec.js`

```javascript
import { test, expect } from '@playwright/test'

test.describe('Document Color Features', () => {
  test('should create document with default color', async ({ page }) => {
    await page.goto('/#/docs')
    await page.click('[data-testid="create-doc-button"]')
    
    const docCard = page.locator('[data-testid="doc-card"]').first()
    await expect(docCard).toBeVisible()
    
    // Check default background (should be white/gray, not colored)
    const bgColor = await docCard.evaluate(el => 
      window.getComputedStyle(el).backgroundColor
    )
    expect(bgColor).toBe('rgba(255, 255, 255, 0.05)')
  })

  test('should change document color via color picker', async ({ page }) => {
    await page.goto('/#/docs')
    
    // Create document
    await page.click('[data-testid="create-doc-button"]')
    const docCard = page.locator('[data-testid="doc-card"]').first()
    
    // Open color picker
    await docCard.hover()
    await page.locator('[title="Color"]').first().click()
    
    // Select red color
    await page.click('button[title="red"]')
    
    // Verify color change
    const bgColor = await docCard.evaluate(el => 
      window.getComputedStyle(el).backgroundColor
    )
    expect(bgColor).toContain('rgb(') // Should be a color, not default
  })

  test('should open document and change color', async ({ page }) => {
    await page.goto('/#/docs')
    
    // Create and open document
    await page.click('[data-testid="create-doc-button"]')
    await page.locator('[data-testid="doc-card"]').first().click()
    
    // Open color picker in editor
    await page.click('[title="Document Color"]')
    
    // Select blue color
    await page.click('button[title="blue"]')
    
    // Go back and verify
    await page.click('[data-testid="back-to-docs"]')
    const docCard = page.locator('[data-testid="doc-card"]').first()
    const bgColor = await docCard.evaluate(el => 
      window.getComputedStyle(el).backgroundColor
    )
    expect(bgColor).toContain('rgb(')
  })

  test('should change color via context menu', async ({ page }) => {
    await page.goto('/#/docs')
    
    // Create document
    await page.click('[data-testid="create-doc-button"]')
    const docCard = page.locator('[data-testid="doc-card"]').first()
    
    // Right-click to open context menu
    await docCard.click({ button: 'right' })
    
    // Select green color from context menu
    await page.click('button[title="green"]')
    
    // Verify color change
    const bgColor = await docCard.evaluate(el => 
      window.getComputedStyle(el).backgroundColor
    )
    expect(bgColor).toContain('rgb(')
  })

  test('should pin and unpin documents', async ({ page }) => {
    await page.goto('/#/docs')
    
    // Create document
    await page.click('[data-testid="create-doc-button"]')
    const docCard = page.locator('[data-testid="doc-card"]').first()
    
    // Hover and click pin button
    await docCard.hover()
    await page.locator('[title="Pin"]').first().click()
    
    // Verify pinned state
    const pinIcon = docCard.locator('[data-testid="pin-icon"]')
    await expect(pinIcon).toHaveAttribute('fill')
    
    // Unpin
    await page.locator('[title="Unpin"]').first().click()
    await expect(pinIcon).not.toHaveAttribute('fill')
  })
})
```

#### 5.3 Manual Testing Checklist

**Color Functionality:**
- [ ] Default color applied to new documents
- [ ] Color picker opens on button click (editor toolbar)
- [ ] Color picker opens on button click (card hover)
- [ ] Color picker opens in context menu
- [ ] All 12 colors are selectable
- [ ] Selected color shows ring indicator
- [ ] Color applies to document card background
- [ ] Color applies to editor background
- [ ] Color persists after page refresh
- [ ] Color persists after reopening document
- [ ] Invalid colors are rejected
- [ ] Color picker closes after selection

**PIN Functionality:**
- [ ] Pin button appears on hover
- [ ] Pin icon toggles between filled/outline
- [ ] Pinned documents show yellow icon background
- [ ] Pin state persists after page refresh
- [ ] Pin button works in both grid and list view
- [ ] Pin button disabled in multi-select mode

**Integration:**
- [ ] Color and PIN can be used together
- [ ] Color picker doesn't interfere with PIN button
- [ ] Context menu shows both options correctly
- [ ] Bulk operations don't affect color/pin settings

---

## UI/UX Considerations

### Color Picker Placement

1. **Editor Toolbar (Primary)**
   - Position: Between title input and save indicator
   - Icon: Palette (lucide-react)
   - Tooltip: "Document Color"
   - Always visible when editing

2. **Card Hover (Secondary)**
   - Position: In actions bar, before Pin button
   - Icon: Palette (lucide-react)
   - Tooltip: "Color"
   - Only visible on hover (same as Pin button)

3. **Context Menu (Tertiary)**
   - Position: Between "Open/Rename" and "Pin"
   - Submenu with grid of colors
   - Right-click alternative

### Visual Design

**Color Picker Grid:**
- 6 columns x 2 rows
- 24px gap between dots
- 24px dot size
- Selected color: 2px accent ring
- Popover container with padding

**Card Background:**
- Transparent when 'default'
- Semi-transparent when colored (0.45 opacity)
- Blend with global transparency setting
- Border stays consistent

**Editor Background:**
- Matches card color
- Slightly higher opacity for readability
- Maintains glassmorphism effect

### Accessibility

- Color picker buttons have proper ARIA labels
- Keyboard navigation for color selection
- Focus indicators on selected color
- Sufficient contrast ratio maintained
- Screen reader announces color changes

### Performance

- Optimistic updates for instant feedback
- API calls debounced if rapid color changes
- Memoized color calculations
- Minimal re-renders on color selection

---

## Migration Strategy

### Database Migration

1. **Backup existing data**
   ```bash
   # Before migration
   cp data/glassy-dash.db data/glassy-dash.db.backup
   ```

2. **Apply migration**
   ```sql
   ALTER TABLE documents ADD COLUMN color VARCHAR(20) DEFAULT 'default';
   UPDATE documents SET color = 'default' WHERE color IS NULL;
   ```

3. **Verify migration**
   ```bash
   # Check schema
   sqlite3 data/glassy-dash.db ".schema documents"
   
   # Verify default values
   sqlite3 data/glassy-dash.db "SELECT id, color FROM documents LIMIT 5"
   ```

4. **Test rollback plan**
   ```sql
   -- If needed, rollback by dropping column
   ALTER TABLE documents DROP COLUMN color;
   ```

### Client-Side Migration

**No breaking changes required** - existing documents will get 'default' color automatically.

**Optional:** Create migration script to suggest colors based on content analysis (future enhancement).

---

## API Documentation Updates

### Endpoints to Document

**POST /api/documents**
```json
{
  "title": "string",
  "content": "string",
  "folderId": "string",
  "color": "string",        // ✅ New - one of: default, red, yellow, green, blue, purple, peach, sage, mint, sky, sand, mauve
  "tags": ["string"]
}
```

**PUT /api/documents/:id**
```json
{
  "title": "string",
  "content": "string",
  "color": "string"         // ✅ New
}
```

**PATCH /api/documents/:id**
```json
{
  "color": "string"         // ✅ New - partial update
}
```

### Response Format

All document endpoints now include color field:
```json
{
  "id": "string",
  "title": "string",
  "content": "string",
  "folderId": "string",
  "color": "string",        // ✅ New
  "tags": ["string"],
  "pinned": boolean,
  "createdAt": "ISO8601",
  "updatedAt": "ISO8601",
  "deletedAt": "ISO8601|null"
}
```

### Error Responses

**Invalid Color:**
```json
{
  "error": "Invalid color"
}
```
Status: 400 Bad Request

---

## Documentation Updates

### Files to Update

1. **glassy-dash/GLASSYDASH/docs/components/DocsView.md**
   - Add color picker section
   - Document color persistence
   - Update props and state documentation

2. **glassy-dash/GLASSYDASH/docs/DATABASE_SCHEMA.md**
   - Add color column to documents table
   - Update with default value

3. **glassy-dash/GLASSYDASH/docs/API_REFERENCE.md**
   - Document new color field
   - Update endpoint examples

4. **glassy-dash/GLASSYDASH/docs/USER_GUIDE.md**
   - Add "Customizing Document Colors" section
   - Include screenshots/color examples

5. **glassy-dash/GLASSYDASH/CHANGELOG.md**
   - Add feature entry
   - Document breaking changes (none expected)

---

## Risk Assessment

### Technical Risks

| Risk | Impact | Probability | Mitigation |
|-------|--------|-------------|------------|
| Database migration fails | High | Low | Backup before migration, test on staging |
| Color not persisting | Medium | Low | Optimistic updates with rollback |
| API validation errors | Medium | Low | Comprehensive validation, error handling |
| Performance degradation | Low | Low | Memoization, debouncing |
| UI conflicts | Medium | Low | Thorough testing of all views |

### User Experience Risks

| Risk | Impact | Probability | Mitigation |
|-------|--------|-------------|------------|
| Confusion about color vs pin | Low | Medium | Distinct icons, tooltips |
| Color picker hard to find | Low | Low | Multiple access points, intuitive placement |
| Performance on many colored docs | Low | Low | Optimized rendering, caching |

---

## Success Criteria

### Functional Requirements

- ✅ Users can assign colors to documents
- ✅ 12 color options available
- ✅ Color persists to database
- ✅ Color displays on document cards
- ✅ Color displays in document editor
- ✅ Color picker accessible from multiple locations
- ✅ PIN functionality remains unchanged
- ✅ No breaking changes to existing features

### Performance Requirements

- ✅ Color selection responds instantly (optimistic update)
- ✅ Page load time unchanged (< 2s)
- ✅ Color picker animation smooth (60fps)
- ✅ No memory leaks with color changes

### Quality Requirements

- ✅ All unit tests pass
- ✅ All E2E tests pass
- ✅ No console errors
- ✅ Accessibility standards met (WCAG 2.1 AA)
- ✅ Consistent with Note Card UX

---

## Rollback Plan

### If Issues Arise

1. **Database Rollback**
   ```sql
   -- Remove color column
   ALTER TABLE documents DROP COLUMN color;
   ```

2. **Code Rollback**
   - Revert docsStore changes
   - Revert DocsView.jsx changes
   - Revert API route changes

3. **Client Rollback**
   - Deploy previous version
   - Clear user localStorage cache
   - Users continue with PIN-only functionality

### Rollback Triggers

- Critical bugs affecting document creation/editing
- Performance degradation > 50%
- Data corruption or loss
- Security vulnerabilities

---

## Timeline

### Development (4-6 hours)

| Phase | Task | Duration |
|--------|-------|----------|
| Phase 1 | Backend & Data Layer | 1.5h |
| Phase 2 | Store & State Management | 1h |
| Phase 3 | UI Components | 2h |
| Phase 4 | Context Menu Integration | 0.5h |
| Phase 5 | Testing | 1h |

### Deployment (1-2 hours)

| Task | Duration |
|-------|----------|
| Code review | 30min |
| Staging deployment | 15min |
| Staging testing | 30min |
| Production deployment | 15min |
| Production verification | 15min |

### Post-Deployment (1 hour)

| Task | Duration |
|-------|----------|
| Monitor logs | 30min |
| User feedback collection | 30min |

**Total Estimated Time:** 6-9 hours

---

## Dependencies

### Required Components
- ✅ ColorDot component (already exists)
- ✅ Popover component (already exists)
- ✅ themes.js utilities (already exists)
- ✅ docsStore (already exists)
- ✅ DocsView component (already exists)

### External Dependencies
- None (uses existing libraries)

### Blocking Issues
- None identified

---

## Future Enhancements

### Phase 2 Features (Post-MVP)

1. **Color Presets**
   - Pre-defined color themes for related documents
   - Folder-level color inheritance

2. **Auto-Coloring**
   - AI-powered color suggestions based on content
   - Tag-based color assignment

3. **Custom Colors**
   - Hex color picker
   - Custom palette management

4. **Color Search/Filter**
   - Filter documents by color
   - Color-based search results

5. **Bulk Color Operations**
   - Color multiple documents at once
   - Color-based bulk actions

---

## Conclusion

This implementation plan provides a comprehensive roadmap for adding color customization features to Document Cards, achieving feature parity with Note Cards. The plan addresses all technical aspects including backend, frontend, testing, and deployment, with careful consideration for user experience, performance, and maintainability.

The phased approach allows for incremental development and testing, reducing risk and ensuring high-quality delivery. Estimated effort of 4-6 hours for development, plus 1-2 hours for deployment and verification, makes this a feasible enhancement that will significantly improve the document organization experience.

---

**Document Version:** 1.0  
**Last Updated:** January 31, 2026  
**Next Review:** After implementation completion