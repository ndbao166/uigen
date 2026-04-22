# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev          # Start dev server (Turbopack)
npm run build        # Production build
npm run lint         # ESLint
npm run test         # Vitest
npm run setup        # Install deps + Prisma generate + migrate
npm run db:reset     # Reset SQLite database
```

Run a single test file: `npx vitest run <path>`

## Environment

`ANTHROPIC_API_KEY` in `.env` is optional — without it, the app falls back to a static mock provider and runs in read-only mode.

## Architecture

UIGen is an AI-powered React component generator with live preview. Users describe components in chat; Claude generates and edits files in a virtual (in-memory) file system; the result renders live in a preview iframe.

**Request flow:**
1. User message → `POST /api/chat` (streaming)
2. Claude (claude-haiku-4-5 via Vercel AI SDK) responds with text + tool calls
3. Tool calls hit `src/lib/tools/str-replace.ts` (edit) and `src/lib/tools/file-manager.ts` (create/delete)
4. `FileSystemContext` (`src/lib/contexts/FileSystemContext`) updates in-memory VFS
5. `src/lib/transform/jsx-transformer.ts` converts JSX → browser-executable JS
6. `PreviewFrame` renders the component live
7. On save, project state (messages + files) persists to SQLite via Prisma server actions

**Key modules:**
- `src/lib/file-system.ts` — `VirtualFileSystem` class, the in-memory file tree
- `src/lib/provider.ts` — wraps real Claude or mock; determines read-only mode
- `src/lib/prompts/generation.tsx` — system prompt that guides Claude's code generation
- `src/lib/auth.ts` — JWT session management (jose + bcrypt)
- `src/lib/anon-work-tracker.ts` — localStorage tracker for anonymous users

**Component layout:**
- `src/components/chat/` — ChatInterface, MessageList, MessageInput, MarkdownRenderer
- `src/components/editor/` — CodeEditor (Monaco), FileTree
- `src/components/preview/` — PreviewFrame
- `src/components/ui/` — Radix-based primitives (Button, Dialog, Tabs, ResizablePanels, etc.)

**Data layer:**
- Prisma + SQLite (`prisma/dev.db`)
- Two models: `User` (email/password) and `Project` (name, userId, messages JSON, data JSON)
- Server actions in `src/actions/` handle project CRUD and auth

**Path alias:** `@/*` → `src/*`

**Node shim:** `node-compat.cjs` is loaded via `NODE_OPTIONS` for Next.js Node.js API compatibility — required for dev and build.
