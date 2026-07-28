# AGENTS.md

This is the authoritative guide for agents working in this repository.

## Project

This repository is a minimal Next.js boilerplate with an MDX blog. Keep the starter small, generic, and ready to extend. Prefer clear boundaries and progressive disclosure over speculative abstractions.

## Constraints

**Git is read-only.** Never commit, push, amend, rebase, or use Git to modify the working tree.

Never commit secrets or place credentials in source.

## Definition of Done

Before claiming a task complete, run these commands and confirm that all exit successfully:

```bash
pnpm test
pnpm run lint
pnpm run format
```

For changes that can affect compilation, routing, or content generation, also run:

```bash
pnpm run build
```

## Architecture

Read `.agents/nextjs-16.instructions.AGENTS.md` before adding routes or changing Next.js 16 App Router code.

### Server-first

Components are React Server Components unless they require state, effects, event handlers, or browser APIs. Keep `"use client"` boundaries at the smallest practical leaf.

### Feature-based structure

Organize code by feature rather than file type:

```text
src/
  app/                          # Next.js routing files only
  content/                      # MDX blog posts
  features/
    blog/
      blog.index.ts             # Public API
      blog.types.ts
      blog.utils.ts
      blog.mdx-components.tsx
      BlogIndex.tsx
      BlogPost.tsx
    header/
      header.index.ts
      Header.tsx
    home/
      home.index.ts
      HomePage.tsx
    theme/
      theme.index.ts
      theme.utils.ts
      AppThemeProvider.tsx
      ThemeSwitch.tsx
  shared/
    shared.index.ts
    PageShell.tsx
```

- Keep feature folders flat.
- Use `<feature>.index.ts` as the only public surface for code outside that feature.
- Prefix feature support files with the feature name.
- Keep `app/` limited to Next.js special routing files.
- Put code used across multiple features or by the app shell in `shared/`.
- Use the `@/*` alias for imports rooted at `src/`.

### Components and exports

Use function declarations with named exports for components. Next.js special files are the only exception because the framework requires default exports.

```tsx
export function Example() {
  return <div />;
}
```

### Composition

Prefer children and app-shell composition over deep prop chains. Keep third-party integration details inside the feature that owns them.

## Stack

- Next.js 16 App Router
- React 19
- TypeScript 5
- Tailwind CSS 4 through PostCSS
- daisyUI 5
- `next-themes`
- Nextra MDX and Nextra Blog styles
- Pagefind
- Vitest
- React Testing Library
- pnpm

## Styling and themes

Read `.agents/frontend-design.instructions.AGENTS.md` before designing or substantially changing pages or components.

Read `.agents/tailwind4-instructions.AGENTS.md` before changing Tailwind CSS.

Tailwind configuration is CSS-first in `src/app/globals.css`; do not add a JavaScript Tailwind configuration file. Use daisyUI semantic tokens rather than hardcoded colors.

The app maps logical `light` and `dark` theme names to the `bumblebee` and `business` daisyUI themes. Keep `theme.utils.ts` and the daisyUI configuration in `globals.css` synchronized.

Fonts are loaded with `next/font/google` in the root layout and exposed through `--font-ubuntu-sans` and `--font-ubuntu-mono`. Do not redeclare them elsewhere.

## Blog

Posts live in `src/content` as MDX and require `title` and ISO-formatted `date` front matter. `description` is optional. Preserve static generation, metadata, newest-first ordering, and Pagefind output when changing the blog.

Nextra loads custom MDX components through the Turbopack alias in `next.config.ts`. Keep that alias synchronized if the MDX component file moves.

## Testing

Co-locate unit and rendered component tests with their feature using `.test.ts` or `.test.tsx`. Use React Testing Library queries that reflect how users and assistive technology find the interface. Mock external framework and content boundaries, not the component behavior under test.

Keep `app/` limited to routing files; page rendering tests belong with the feature that supplies the page UI.

## Workflows

```bash
pnpm dev
pnpm build
pnpm start
pnpm test
pnpm test:watch
pnpm lint
pnpm format
```

ESLint uses flat configuration. Prettier sorts Tailwind utility classes through `prettier-plugin-tailwindcss`.

## Additional references

| File                                             | Read before                                              |
| ------------------------------------------------ | -------------------------------------------------------- |
| `.agents/frontend-design.instructions.AGENTS.md` | Designing or substantially changing pages and components |
| `.agents/nextjs-16.instructions.AGENTS.md`       | Adding routes or changing App Router code                |
| `.agents/tailwind4-instructions.AGENTS.md`       | Changing Tailwind CSS or theme configuration             |
