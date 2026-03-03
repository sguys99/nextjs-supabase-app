# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev        # Start development server (localhost:3000)
npm run build      # Production build
npm run lint       # ESLint check
```

Add shadcn/ui components:
```bash
npx shadcn@latest add <component-name>
```

## Architecture

This is a **Next.js 15 App Router** project with **Supabase** for authentication and database, using **React 19**, **Tailwind CSS**, and **shadcn/ui** (new-york style).

### Route Structure

- `/` — Public landing/home page (`app/page.tsx`)
- `/auth/*` — Authentication routes (login, sign-up, forgot-password, update-password, confirm, error)
- `/protected/*` — Auth-gated routes with shared layout (`app/protected/layout.tsx`)

### Authentication Flow

Auth is enforced by `lib/supabase/proxy.ts` (session refresh middleware), wired into `proxy.ts` at the project root. Any route outside `/`, `/login`, or `/auth/*` redirects unauthenticated users to `/auth/login`.

Three Supabase client factories serve different contexts:
- `lib/supabase/client.ts` — Browser client (`createBrowserClient`)
- `lib/supabase/server.ts` — Server Components and Route Handlers (`createServerClient` with cookie store)
- `lib/supabase/proxy.ts` — Proxy/middleware (`createServerClient` with request cookies)

**Critical**: Never store server clients in global variables — always instantiate within the function scope (required for Fluid compute compatibility).

### Key Conventions

**Server Components by default** — only add `'use client'` when using state, effects, or event handlers.

**Always `await` async request APIs** (Next.js 15):
```typescript
// params and searchParams are now Promises
const { id } = await params
const cookieStore = await cookies()
```

**Path aliases** (defined in `components.json`):
- `@/components` — React components
- `@/components/ui` — shadcn/ui base components
- `@/lib` — utilities and Supabase clients
- `@/hooks` — custom hooks

**Styling rules**:
- Use Tailwind utility classes exclusively — no inline styles, no CSS modules
- Use `cn()` from `@/lib/utils` for conditional class merging
- Use semantic color variables (`bg-background`, `text-foreground`, `text-muted-foreground`) — never hardcode colors like `bg-white` or `text-gray-900`
- Dark mode is handled automatically via CSS variables and `next-themes`

**Naming**:
- Files: `kebab-case.tsx`
- Components: `PascalCase` with named exports (default export only for page components)
- Folders: `lowercase` or `kebab-case`

**Environment variables** required in `.env.local`:
```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=
```

## Development Guides

Detailed guides are in `docs/guides/`:
- `nextjs-15.md` — Next.js 15 patterns, async APIs, Server Actions, caching
- `component-patterns.md` — Server/Client boundaries, CVA variants, compound components
- `styling-guide.md` — Tailwind + shadcn/ui usage, color system, dark mode
- `project-structure.md` — Folder organization, naming conventions, import ordering

## Custom Agents & Commands

Project-specific Claude agents are in `.claude/agents/`:
- `dev/` — `nextjs-app-developer`, `ui-markup-specialist`, `code-reviewer`, `starter-cleaner`, `development-planner`
- `docs/` — `prd-generator`, `prd-validator`
- `notion-api-expert.md`

Custom slash commands in `.claude/commands/`:
- `/git:commit`, `/git:branch`, `/git:merge`, `/git:pr`
- `/docs:update-roadmap`
