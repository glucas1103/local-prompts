# Markdown Editor - Triple Panel Editor with AI

## Overview

The Markdown Editor domain is a **feature in development** (User Story 1.1) that will allow editing Markdown documentation with the help of an AI assistant.

**Status**: 📝 PLANNED (Draft)  
**Coverage**: 10% (spec only)

---

## Planned Architecture

### Left Panel - File Explorer

- **Technology**: react-arborist
- **Features**: File/folder tree, CRUD, drag & drop

### Center Panel - Content Editor

- **Technology**: TipTap (Markdown editor)
- **Features**: Real-time editing, auto-save (debounce 300ms)

### Right Panel - AI Chat

- **Technology**: ❓ AI backend to be defined
- **Features**: Chat with assistant, AI actions (modify/create/delete files)

### Panels Management

- **Technology**: react-resizable-panels
- **Features**: Resizing, localStorage save

### State Management

- **Local UI**: Zustand
- **Server State**: TanStack Query

---

## User Story

**Complete reference**: `/roadmap/user-stories/user-story-1.1-frontend-triple-panel.md`

**Story status**: Draft  
**Estimation points**: 13 points (epic, to be broken down)

---

## Unresolved Questions

### Storage

❓ **Where will Markdown files be stored?**

- Option A: PostgreSQL
- Option B: S3 or cloud storage
- Option C: GitHub directly (automatic commits)
- Option D: Other

### AI Backend

❓ **Which AI service will be used?**

- Option A: OpenAI (GPT-4, GPT-3.5)
- Option B: Anthropic (Claude)
- Option C: Other LLM
- Option D: To be defined

### Versioning

❓ **Will there be file history?**

- Integrated versioning?
- Snapshots?
- Git-like system?

### Collaboration

❓ **Real-time collaborative editing?**

- Multi-user editing?
- Conflict resolution?
- Cursors/presence?

---

## Current Implementation

❌ **No code found** in the current codebase

**Searched**:

- `apps/web/src/app/(protected)/dashboard/repos/[repositoryId]/markdown-editor/`
- Components `triple-panel`, `file-explorer`, `ai-chat`
- API routes for markdown

**Result**: Not implemented

---

## Next Steps

1. **Product Phase** (ProductMan): Validate need and use cases
2. **Spec Phase** (SpecMan): Define technical architecture, storage/AI choices
3. **Dev Phase** (DevMan): Implementation according to US-1.1

---

## References

- `/roadmap/user-stories/user-story-1.1-frontend-triple-panel.md` - Complete spec (385 lines)
- Potential User Story: `user-story-1.2-backend-markdown-api.md` (not found)

---

**Last updated**: 2025-10-17  
**Status**: 📝 PLANNED - Not yet implemented
