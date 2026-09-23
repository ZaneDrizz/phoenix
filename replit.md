# Workspace

## Overview

pnpm workspace monorepo using TypeScript. Each package manages its own dependencies.

## Stack

- **Monorepo tool**: pnpm workspaces
- **Node.js version**: 24
- **Package manager**: pnpm
- **TypeScript version**: 5.9
- **API framework**: Express 5
- **Database**: PostgreSQL + Drizzle ORM
- **Validation**: Zod (`zod/v4`), `drizzle-zod`
- **API codegen**: Orval (from OpenAPI spec)
- **Build**: esbuild (CJS bundle)
- **HTML parsing**: cheerio (server-side proxy rewriting)
- **Auth**: bcryptjs password hashing + UUID tokens stored in DB

## Key Commands

- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from OpenAPI spec
- `pnpm --filter @workspace/api-server run dev` — run API server locally

## Artifacts

### Web Proxy (`artifacts/proxy-site`)
- React + Vite frontend at `/`
- Full onboarding: `login → welcome → theme → cloak → autocloak → home`
- 44 themes with visual swatch pickers, 8 particle types (including shooting stars)
- Sidebar nav: Home, Games, Movies, TV, Music (Spotify), AI, Apps, Friends (chat), Settings, Incognito

### API Server (`artifacts/api-server`)
- Express 5 backend at `/api`
- `/api/proxy/fetch` — POST: fetch URL through proxy (uses cheerio for HTML rewriting)
- `/api/proxy/history` — GET: browse history, DELETE: clear history
- `/api/healthz` — health check
- **Auth** (Username + Password only):
  - `POST /api/auth/register` — create account (username: 2-24 chars, alphanumeric+underscore; password: min 4 chars)
  - `POST /api/auth/login` — sign in, returns `{username, token}`
  - `GET /api/auth/me` — validate token (Bearer)
- **Global Chat** (24/7 persistent):
  - `GET /api/chat/messages` — last 80 messages (requires Bearer token)
  - `POST /api/chat/message` — send message (requires Bearer token, max 500 chars)
- Movies & TV (TMDb-backed; uses `TMDB_API_KEY` env var if set, else public fallback key):
  - `/api/movies/discover`, `/api/movies/trending`, `/api/movies/genres`, `/api/movies/genre/:id`
  - `/api/movies/search?q=…`, `/api/movies/details/:id`
  - `/api/tv/discover`, `/api/tv/details/:id`, `/api/tv/season/:id/:season`

## Frontend Stages (proxy-site)
- `login` — Username/Password sign-in / create account (gate before all other stages)
- `welcome → theme → cloak → autocloak → home` — onboarding (runs once after first login)
- `home`, `browser`, `games`, `movies`, `chat` — main app views

## Database Schema

- `proxy_history` — proxied URL history (id, url, finalUrl, title, favicon, visitedAt)
- `users` — accounts (id, username UNIQUE, password_hash, token, created_at)
- `chat_messages` — global chat (id, username, content, created_at)

## Auth Flow
- Token stored in `localStorage` as `phoenix.token`; username as `phoenix.username`
- All chat API calls send `Authorization: Bearer <token>`
- No token → `login` stage shown; invalid token → redirect to login on next refresh

See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details.
