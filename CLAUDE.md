# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Claudia is a desktop application for Claude Desktop that provides advanced session management, analytics, and developer tools. It's built with Tauri 2 (Rust backend) and React 18 (TypeScript frontend).

## Development Commands

**Important: This project uses Bun, not npm or yarn.**

```bash
# Install dependencies
bun install

# Run development mode (opens Tauri window with hot reload)
bun run dev

# Alternative: Run Tauri dev directly
bun run tauri dev

# Build for production
bun run build

# Run Rust tests (61 tests covering core functionality)
cargo test

# Check Rust code
cargo check
cargo clippy
```

## Architecture

### Frontend (`/src`)
- **Framework**: React 18 + TypeScript + Vite 6
- **Styling**: Tailwind CSS v4 with custom theme
- **State**: React hooks + Context API
- **API Client**: `/src/lib/api.ts` - Complete TypeScript client for backend

### Backend (`/src-tauri`)
- **Framework**: Tauri 2 with Rust
- **Database**: SQLite via rusqlite
- **Key Modules**:
  - `db/` - Database operations for sessions, projects, agents
  - `utils/` - Claude Desktop integration, process management
  - `api/` - REST API server implementation
  - `security/` - OS-level sandboxing (seccomp/Seatbelt)

### Key Features

1. **Project & Session Management**
   - Visual browser for `~/.claude/projects/`
   - Checkpoint system with timeline visualization
   - Session forking and comparison

2. **CC Agents** (`/src-tauri/src/utils/cc_agents.rs`)
   - Custom AI agents with sandboxed execution
   - Real-time JSONL streaming
   - Process registry with lifecycle management

3. **Security** (`/src-tauri/src/security/`)
   - Platform-specific sandboxing:
     - Linux: seccomp-based restrictions
     - macOS: Seatbelt profiles
   - Prevents network access, file system restrictions

4. **Analytics** (`/src/components/analytics/`)
   - Token usage tracking
   - Cost calculation
   - Usage patterns visualization

## Common Development Tasks

### Adding a New API Endpoint
1. Add handler in `/src-tauri/src/api/handlers/`
2. Register route in `/src-tauri/src/api/server.rs`
3. Add TypeScript types and client method in `/src/lib/api.ts`
4. Use in React components via the API client

### Working with the Database
- Schema: `/src-tauri/migrations/`
- Models: `/src-tauri/src/db/models.rs`
- Operations: `/src-tauri/src/db/` (projects.rs, sessions.rs, etc.)

### Adding UI Components
- Components: `/src/components/`
- Follow existing patterns (SessionCard, ProjectCard examples)
- Use shadcn/ui components from `/src/components/ui/`
- Tailwind v4 for styling

### Testing
- Backend: `cargo test` (comprehensive test suite)
- Frontend: No test setup currently
- Manual testing: Use `bun run dev` for hot reload

## Troubleshooting

### Build Issues
- Ensure Bun is installed (not npm/yarn)
- Check Rust toolchain: `rustup update`
- Platform-specific issues documented in README

### API Connection
- Backend runs on port from `/src-tauri/src/api/server.rs`
- Frontend API client auto-configures in development
- Check Tauri IPC commands in `/src-tauri/src/main.rs`

### Database
- Location: `~/.claudia/claudia.db`
- Reset: Delete the file to start fresh
- Migrations run automatically on startup

## Important Implementation Details

### Process Management
- Registry pattern in `/src-tauri/src/utils/process_registry.rs`
- Graceful shutdown with timeout handling
- Real-time output streaming via JSONL

### Claude Desktop Integration
- Reads from `~/.claude/` directory structure
- Parses SQLite databases and JSON files
- Non-invasive (read-only access)

### State Management
- Frontend: React Context + hooks
- Backend: Tauri state + Arc<Mutex<>> for shared state
- Session state persisted to SQLite

### Error Handling
- Comprehensive error types in `/src-tauri/src/errors.rs`
- User-friendly error messages in UI
- Detailed logging for debugging