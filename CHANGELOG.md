# 📝 Changelog

All notable changes to GLASSYDASH are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [1.1.6] - 2026-01-28

### Added

- 🚀 **GlassyDocs System** - Full document management with nested folders and hierarchy
- 🚀 **Voice Studio** - Complete audio recording suite with AI transcription
- 🚀 **Multimedia Support** - YouTube integration and self-hosted music streaming
- 🚀 **Admin Dashboard** - User management, storage quotas, and system monitoring
- 🚀 **Enhanced Theming** - Theme presets, custom backgrounds, and accent colors
- 🚀 **Robust Error Handling** - Network resilience with automatic retry logic

### Fixed

- 🐛 Note card overflow and scrolling issues
- 🐛 Network failures with automatic retry (3 attempts)
- 🐛 Authentication errors with session expiration handling
- 🐛 Audio errors with microphone access troubleshooting
- 🐛 API errors with validation messages
- 🎨 UI consistency across Docs, Voice, and Notes views

### Planned Features

- Multi-language support (Spanish, Chinese)
- Enhanced AI with larger models
- Mobile app (React Native)
- Plugin system
- Advanced collaboration features

---

## [0.67.0] - 2026-01-23 (Beta)

### Added

- 🎨 **Advanced Theming System**
  - Theme presets (Neon Tokyo, Zen Garden, etc.)
  - Custom background library (Mobile/Desktop/4K optimized)
  - 7 bioluminescent accent colors
  - Card transparency levels (5 options)
  - Smart overlay for readability

- 🤖 **Private AI Assistant (Llama 3.2)**
  - 100% local and private AI integration
  - Note-aware RAG capabilities
  - Smart search and question answering
  - No data leaves your server

- 👥 **Real-time Collaboration**
  - Live note collaboration
  - Real-time checklist sync
  - Add/remove collaborators
  - View-only mode
  - Automatic conflict resolution

- 📝 **Drawing Notes**
  - Freehand drawing canvas
  - Customizable brush sizes
  - Color palette selection
  - Drawing preview

- 📌 **Enhanced Note Features**
  - Pin/unpin functionality
  - Drag-and-drop reordering
  - Bulk operations (delete, pin, color)
  - Tag chip management

- 🔍 **Advanced Search**
  - Full-text search (title, content, tags, checklist, images)
  - AI-powered query assistance
  - Quick filters (Notes, All Images)

- 💾 **Import/Export**
  - Export all notes to JSON
  - Import JSON (merge support)
  - Per-note Markdown export
  - Google Keep import (Takeout format)

- 👨‍💼 **Admin Panel**
  - User management (CRUD)
  - Password reset
  - Admin role assignment
  - Registration toggle
  - User statistics

- 🔐 **Security Enhancements**
  - JWT-based authentication
  - Secret key recovery
  - Password hashing (bcrypt)
  - Protected API endpoints

- 🎯 **UI Improvements**
  - Responsive design (mobile-first)
  - Dark/Light mode
  - Glassmorphism design
  - Improved color picker
  - Emoji icons for note types

### Changed

- 🔄 **Architecture Refactor**
  - Migrated to React 18
  - Updated to Vite 5.x
  - Express.js 4.x backend
  - SQLite with better-sqlite3

- 📱 **PWA Support**
  - Offline capability
  - Installable on desktop/mobile
  - Service worker implementation

### Fixed

- 🐛 Bug: Checklist items not persisting on save
- 🐛 Bug: Tag chips not rendering properly on overflow
- 🐛 Bug: Images not compressing on upload
- 🐛 Bug: Search not finding checklist items
- 🐛 Bug: Modal not closing on Escape key
- 🐛 Bug: Drag-and-drop not working on mobile

### Security

- 🔒 Security: Updated JWT secret handling
- 🔒 Security: Added CORS protection
- 🔒 Security: Enhanced password hashing
- 🔒 Security: SQL injection prevention

### Performance

- ⚡ Performance: Optimized image compression
- ⚡ Performance: Reduced bundle size by 40%
- ⚡ Performance: Debounced search input
- ⚡ Performance: Virtualized note list (in progress)

### Documentation

- 📚 Documentation: Comprehensive user guides
- 📚 Documentation: Developer documentation
- 📚 Documentation: API reference
- 📚 Documentation: Troubleshooting guide
- 📚 Documentation: Quick start guide

---

## [0.66.0] - 2026-01-15

### Added

- ✨ Markdown support with formatting toolbar
- ✨ Checklists with drag-to-reorder
- ✨ Image attachments with compression
- ✨ Tag system with chips
- ✨ Per-note color themes
- ✨ Server-Sent Events (SSE) for real-time updates

### Fixed

- 🐛 Bug: Notes not syncing across devices
- 🐛 Bug: Login session expiring too quickly

---

## [0.65.0] - 2026-01-08

### Added

- ✨ Multi-user authentication system
- ✨ User registration
- ✨ Admin panel basic features
- ✨ Database schema with migrations

### Changed

- 🔄 Migrated from local storage to SQLite
- 🔄 Implemented user isolation

---

## [0.60.0] - 2026-01-01

### Added

- ✨ Initial release of GLASSYDASH
- ✨ Basic note creation
- ✨ Markdown editor
- ✨ Local storage persistence
- ✨ Simple UI with glassmorphism

---

## Versioning Policy

GLASSYDASH follows [Semantic Versioning 2.0.0](https://semver.org/):

- **MAJOR** version for incompatible API changes
- **MINOR** version for backwards-compatible functionality additions
- **PATCH** version for backwards-compatible bug fixes

### Beta Releases

Beta versions (0.x.x) may have:

- Breaking changes
- Unstable features
- Experimental functionality

**Stable release**: 1.0.0 planned for Q2 2026

---

## Categories

### Added

New features and enhancements

### Changed

Changes to existing functionality

### Deprecated

Soon-to-be removed features

### Removed

Removed features

### Fixed

Bug fixes

### Security

Security updates and fixes

### Performance

Performance improvements

### Documentation

Documentation updates

---

## Migration Guide

### From v0.66 to v0.67

No breaking changes. Just upgrade:

```bash
git pull origin main
npm install
npm run build
npm start
```

### From v0.65 to v0.66

Database migration required:

```bash
npm run migrate
```

---

## Release Process

1. Update version in package.json
2. Update CHANGELOG.md
3. Create git tag
4. Run tests
5. Build release
6. Create GitHub release
7. Publish to npm (if applicable)
8. Deploy to Docker Hub

---

**For Older Releases**
See [GitHub Releases](https://github.com/yourusername/glassy-dash/releases)

**Current Version**: 1.1.6  
**Last Updated**: January 28, 2026
