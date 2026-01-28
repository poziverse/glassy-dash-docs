# Component Specifications & Regression Prevention

This document outlines strict implementation rules for critical components. These rules **must** be followed during future development to prevent recurring regressions.

## 1. NoteCard Component (`src/components/NoteCard.jsx`)

### 1.1 Interaction Logic (Click Handling)

**Requirement**: The card must be clickable to open, but interactive elements inside it (buttons, links, inputs) must **not** trigger the open action.
**Critical Implementation Rule**: The "Self-Blocking" Check.
The card element itself often has the `.cursor-pointer` class. A naive `closest('.cursor-pointer')` check will find the card itself and block the click.
**CORRECT LOGIC**:

```javascript
// DO NOT use: if (e.target.closest('.cursor-pointer')) return;
// USE THIS INSTEAD:
const interactive = e.target.closest(
  'button, input, a, .cursor-pointer, [role="button"]',
);
// Allow the click if the "interactive" element is the card container itself
if (interactive && interactive !== e.currentTarget) {
  return;
}
```

### 1.2 Display & Layout

**Requirement**: Note cards must display their full text content in the dashboard view.
**Restricted CSS**:

- **NEVER** apply `line-clamp` or fixed `max-height` to `.note-content` within `.note-card`.
- The Masonry layout handles variable heights. Truncating text hides valuable information and breaks the user experience.

### 1.3 Z-Index & Layers

**Requirement**: Interactive elements (Pack, Pin, Menu) must be accessible above the card body.

- **Card Container**: `relative`, `z-0` (implicit).
- **Interactive Overlays**: `absolute`, `z-10` (or higher if needed).
- **Click Propagation**: All overlay buttons must call `e.stopPropagation()` to prevent opening the card modal.

---

## 2. Note Editor Modal (`src/components/Modal.jsx`)

### 2.1 Textarea Auto-Resizing

**Requirement**: The editor text area must **always** expand to fit the full content immediately upon opening.
**Regression Risk**: The textarea defaults to a small `min-height` with `overflow-hidden`. If the resize logic only runs on `input` events, the user will see cut-off text when first opening a long note.
**Mandatory Hook**:

```javascript
// MUST trigger resize on mount/mode switch
React.useEffect(() => {
  if (open && viewMode === "edit" && mType === "text") {
    requestAnimationFrame(() => {
      resizeModalTextarea();
    });
  }
}, [open, viewMode, mType, mBody]);
```

### 2.2 Z-Index Layering

- **Scrim/Backdrop**: `z-40` (Fixed).
- **Modal Card**: Child of Scrim.
- **Sticky Header**: `z-20` (Must be above scrolling body content).
- **Popovers/Dropdowns**: `z-[99999]` (Portal-based) or `z-50` (Inline).
- **Toast Notifications**: `z-50` (Global).

### 2.3 Autosave Behavior

**Requirement**: Closing the modal by any means (Close button, Scrim click, Escape key) **must** trigger a save if there are unsaved changes.
**Interaction**: User should not be prompted to save; it should be automatic.
**Key Bindings**:

- `Enter`: Inserts a newline (Standard behavior).
- `Ctrl + Enter`: Explicit save and close (Optional shortcut).

---

## 3. General "Safe Markdown" Rules

**File**: `src/utils/safe-markdown.js`

- **Link Handling**: All links must open in a new tab (`target="_blank"`) to prevent data loss in the editor.
- **Sanitization**: `DOMPurify` is mandatory. Never bypass it for user-generated content.

---

## 4. Motion & Animation Standards

**Library**: `framer-motion` (Mandatory for interaction animations)

### 4.1 Fluid Motion (Spring Physics)

**Requirement**: All interactive elements (cards, modals) must use physics-based spring animations, not linear CSS transitions, to provide a "tactile" and "fluid" feel.

**Standard Spring Config**:
```javascript
export const springConfig = {
  type: "spring",
  stiffness: 300,
  damping: 30,
};
```

---

## 5. Layout & Navigation (`src/components/DashboardLayout.jsx`)

### 5.1 Global Navbar Actions

**Requirement**: Views must be able to inject utility buttons or telemetry into the primary top navigation bar.
**Implementation**: Use the `headerActions` prop on `DashboardLayout`.
**Example**:
```jsx
<DashboardLayout
  title="Mission Control"
  headerActions={<div className="uptime-telemetry">...</div>}
>
  {/* Content */}
</DashboardLayout>
```

### 5.2 Sidebar-Aware Alignment

**Requirement**: Fixed elements anchored to the bottom or sides of the main content area (like `PageHeaderBottom`) must stay aligned with the content regardless of sidebar state.
**CSS Variable**: The project uses a dynamic `--sidebar-width` CSS variable set on the `.main-layout` or `root`.
- **Expanded**: 256px
- **Collapsed**: 72px

**Usage**:
```css
.bottom-floating-header {
  left: calc(var(--sidebar-width) + 1.5rem);
}
```

---

## 6. Development Context (Phase 6+)

### 6.1 View Cleanup

**Requirement**: Secondary headers (`PageHeaderBottom`) should only be used when they provide unique view-specific context that cannot fit in the top navbar.
**Current State**: Removed redundant headers from `AdminView`, `HealthView`, `TrashView`, `SettingsView`, and `DocsView` to reduce visual noise. Titles are now managed by `DashboardLayout`.

```javascript
transition: { type: "spring", stiffness: 400, damping: 17 }
```

### 4.2 Shared Element Transitions

**Requirement**: Modals should appear to "expand" from their origin card.

- **Modal Entry**: Use `initial={{ scale: 0.9 }}` -> `animate={{ scale: 1 }}` to mimic expansion.
- **Card Hover**: Cards must lift subtly on hover (`scale: 1.02`) to indicate interactivity.

---

## 5. Visual Design Specs

### 5.1 Liquid Glow

**Requirement**: "Glass" elements must feature a dynamic "Liquid Glow" effect.

- **Implementation**: `box-shadow` and `border-color` must change on hover to simulate a shifting light source.
- **Variable**: Use `--color-accent-glow` for the highlight color.
