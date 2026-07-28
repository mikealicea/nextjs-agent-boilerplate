# Next.js Agent Boilerplate

A small, opinionated Next.js starter with a feature-based architecture and an MDX blog.

## Included

- Next.js 16 App Router and React 19
- TypeScript 5
- Tailwind CSS 4 and daisyUI 5
- Light and dark themes with `next-themes`
- Nextra-powered MDX posts and Pagefind indexing
- Vitest for unit tests
- React Testing Library, ESLint, and Prettier
- Repository guidance for coding agents

## Start the app

```bash
pnpm install
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000).

## Project structure

```text
src/
  app/                 # Routing files only
  content/             # MDX blog posts
  features/            # Flat, self-contained feature slices
  shared/              # App-wide components
```

Each feature exposes a single `<feature>.index.ts` public API. Code outside a feature imports from that file rather than reaching into feature internals.

## Add a post

Create an MDX file in `src/content`:

```mdx
---
title: "Post title"
date: "2026-07-28"
description: "A short description."
---

Post content.
```

The filename becomes the route under `/blog`.

## Commands

```bash
pnpm dev       # Start the development server
pnpm build     # Build the app and generate the Pagefind index
pnpm start     # Serve the production build
pnpm test      # Run unit tests once
pnpm test:watch # Run unit tests in watch mode
pnpm lint      # Run ESLint
pnpm format    # Format the repository
```

## Agent guidance

Repository conventions live in `AGENTS.md`, with focused references for [frontend design](.agents/frontend-design.instructions.AGENTS.md), [Next.js 16](.agents/nextjs-16.instructions.AGENTS.md), and [Tailwind CSS 4](.agents/tailwind4-instructions.AGENTS.md).
