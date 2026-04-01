# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies, generate Prisma client, and run migrations
npm run setup

# Development server (with turbopack)
npm run dev

# Build for production
npm run build

# Lint
npm run lint

# Run tests
npm test

# Run a single test file
npx vitest run src/path/to/test.test.ts

# Reset the database
npm run db:reset
```

## Architecture

UIGen is an AI-powered React component generator. Users describe components in natural language and Claude generates them in real-time with live preview.

### Request Flow

```
Chat input → ChatInterface
  → POST /api/chat (streaming)
    → Claude (claude-haiku-4-5) via Vercel AI SDK
      → Tools: str_replace_editor / file_manager
        → Virtual File System (in-memory)
          → PreviewFrame (Babel JSX → live iframe)
            → Database (Prisma/SQLite) — authenticated users only
```

### Key Directories

- `src/app/` — Next.js App Router pages and API routes
- `src/app/api/chat/route.ts` — Main streaming endpoint; handles Claude tool calls and project persistence
- `src/components/` — UI split into `chat/`, `editor/`, `preview/`, `auth/`
- `src/lib/` — Core logic: `file-system.ts`, `provider.ts`, `transform/`, `prompts/`
- `src/actions/` — Server actions for project and user CRUD
- `prisma/schema.prisma` — SQLite schema: `User` and `Project` models

### Virtual File System

`src/lib/file-system.ts` manages an **in-memory** file tree — nothing is written to disk. Files are serialized into the `Project.data` JSON column when saved.

### AI Provider

`src/lib/provider.ts` abstracts the AI layer:
- With `ANTHROPIC_API_KEY` set: uses Claude Haiku 4.5 via `@ai-sdk/anthropic`
- Without it: falls back to a mock provider that returns static responses

The system prompt lives in `src/lib/prompts/generation.tsx`.

### JSX Preview

`src/lib/transform/jsx-transformer.ts` transforms generated code into runnable browser JS using Babel standalone. `PreviewFrame` renders it in an iframe with an import map for dependencies.

### Authentication

JWT via `jose`, stored in HTTP-only cookies. Middleware protects `/api/projects` and `/api/filesystem`. Anonymous users can still generate components; their work is only persisted if authenticated.

## Environment

Copy `.env.example` to `.env` and set:
- `ANTHROPIC_API_KEY` — optional; enables real AI generation
- `JWT_SECRET` — required for auth

## Code Style

- Use comments sparingly — only for complex or non-obvious logic.

## Tech Stack

- Next.js 15 (App Router) + React 19 + TypeScript
- Tailwind CSS v4 + shadcn/ui (new-york style, Radix primitives)
- Monaco Editor for code editing
- Prisma + SQLite for persistence
- Vitest + jsdom + Testing Library for tests
