---
name: nextjs-16-instructions
description: Read before adding routes or working with Next.js 16 App Router patterns.
---

# Next.js 16 App Router Essentials

## Prerequisites

- **React 19+** required (project uses 19.2.3 ✓)
- **Node.js 20.9+** minimum
- **TypeScript 5.1+** minimum

## Async Request APIs (Breaking Change)

All dynamic request APIs are now **async-only**. Must use `await`.

### In Pages

```tsx
// app/blog/[slug]/page.tsx
export default async function Page(props: {
  params: Promise<{ slug: string }>;
  searchParams: Promise<{ q?: string }>;
}) {
  const { slug } = await props.params;
  const searchParams = await props.searchParams;
  const query = searchParams.q;

  return <h1>Post: {slug}</h1>;
}
```

### In Layouts

```tsx
// app/blog/[category]/layout.tsx
export default async function Layout(props: {
  children: React.ReactNode;
  params: Promise<{ category: string }>;
}) {
  const { category } = await props.params;
  return <div data-category={category}>{props.children}</div>;
}
```

### In Route Handlers

```tsx
// app/api/posts/route.ts
export async function GET(
  request: Request,
  context: { params: Promise<{ id: string }> },
) {
  const { id } = await context.params;
  return Response.json({ id });
}
```

### In Server Actions

```ts
"use server";
import { cookies, headers } from "next/headers";

export async function action() {
  const cookieStore = await cookies();
  const token = cookieStore.get("token");

  const headersList = await headers();
  const userAgent = headersList.get("user-agent");
}
```

## Metadata API

### Static Metadata (Current Pattern)

```tsx
// app/layout.tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "Hello world",
  description: "A minimal Next.js boilerplate with an MDX blog.",
};
```

### Dynamic Metadata

```tsx
// app/blog/[slug]/page.tsx
import type { Metadata } from "next";

export async function generateMetadata(props: {
  params: Promise<{ slug: string }>;
}): Promise<Metadata> {
  const { slug } = await props.params;
  const post = await fetchPost(slug);

  return {
    title: post.title,
    description: post.excerpt,
  };
}
```

### Type Helpers (npx next typegen)

```tsx
// After running: npx next typegen
import type { PageProps } from "@/.next/types/app/blog/[slug]/page";

export default async function Page(props: PageProps) {
  const { slug } = await props.params; // Fully typed
}
```

## Server vs Client Components

### Default: Server Components

All components are **Server Components** by default. Can:

- Access backend directly (DB, filesystem)
- Use async/await at component level
- Zero JavaScript shipped to client

**Current project examples:**

- `app/layout.tsx` — Server Component
- `app/page.tsx` — Server Component

### When to Use Client Components

Add `"use client"` directive when component needs:

- **State**: `useState`, `useReducer`
- **Effects**: `useEffect`, `useLayoutEffect`
- **Browser APIs**: `window`, `document`, `localStorage`
- **Event handlers**: `onClick`, `onChange`, `onSubmit`
- **Context**: `useContext` (consumer side)
- **Third-party hooks**: `useTheme`, `useRouter` (from `next/navigation`)

**Current project examples:**

- `src/features/theme/ThemeSwitch.tsx` — Client (needs `useTheme` hook)

### Best Practices

- Keep client boundaries **minimal** (don't convert entire page trees)
- Pass serializable props only (no functions, Date objects, etc.)
- Server Components can import Client Components, not vice versa

## Font Optimization (Current Pattern)

```tsx
// app/layout.tsx
import { Ubuntu, Ubuntu_Mono } from "next/font/google";

const ubuntuSans = Ubuntu({
  subsets: ["latin"],
  weight: ["300", "400", "500", "700"],
  variable: "--font-ubuntu-sans",
});

const ubuntuMono = Ubuntu_Mono({
  subsets: ["latin"],
  weight: ["400", "700"],
  variable: "--font-ubuntu-mono",
});

export default function RootLayout({ children }) {
  return (
    <html className={`${ubuntuSans.variable} ${ubuntuMono.variable}`}>
      <body className="font-sans">{children}</body>
    </html>
  );
}
```

Access via Tailwind: `font-sans` (maps to `--font-ubuntu-sans`), `font-mono` (maps to `--font-ubuntu-mono`)

## Caching APIs (Now Stable)

Remove `unstable_` prefix from all cache APIs.

### cacheLife

```ts
import { unstable_cacheLife as cacheLife } from "next/cache"; // ❌ Old
import { cacheLife } from "next/cache"; // ✅ New

export async function getArticles() {
  "use cache";
  cacheLife("hours");
  return await db.articles.findMany();
}
```

### cacheTag

```ts
import { cacheTag } from "next/cache";

export async function getUser(id: string) {
  "use cache";
  cacheTag(`user-${id}`);
  return await db.users.findById(id);
}
```

### revalidateTag

```ts
"use server";
import { revalidateTag } from "next/cache";

export async function updateArticle(id: string) {
  await db.articles.update(id, data);
  revalidateTag(`article-${id}`, "max"); // Stale-while-revalidate
}
```

### updateTag

```ts
"use server";
import { updateTag } from "next/cache";

export async function updateProfile(userId: string, profile: Profile) {
  await db.users.update(userId, profile);
  updateTag(`user-${userId}`); // Read-your-writes (immediate refresh)
}
```

**When to use:**

- `revalidateTag`: Content where delay is acceptable (blog posts, catalogs)
- `updateTag`: Interactive features requiring immediate feedback (forms, settings)

## Configuration

### next.config.ts (Current Pattern)

```ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  // Config options
};

export default nextConfig;
```

### ESLint Flat Config (Current Pattern)

Already migrated to `eslint.config.mjs` ✓

## Common Patterns

### Data Fetching in Server Components

```tsx
async function fetchData() {
  const res = await fetch("https://api.example.com/data", {
    next: { revalidate: 3600 }, // ISR: revalidate every hour
  });
  return res.json();
}

export default async function Page() {
  const data = await fetchData();
  return <div>{data.title}</div>;
}
```

### Loading States

```tsx
// app/blog/loading.tsx
export default function Loading() {
  return <div>Loading...</div>;
}
```

### Error Boundaries

```tsx
// app/blog/error.tsx
"use client"; // Error components must be Client Components

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}
```

## Not Relevant to This Project

- **Middleware/Proxy**: No middleware file exists
- **Turbopack**: Already enabled by default in v16
- **Custom Image Config**: Using defaults (no config needed)
- **PPR**: Not enabled (requires `cacheComponents: true`)

---

_Reference: [Next.js 16 Upgrade Guide](https://nextjs.org/docs/app/guides/upgrading/version-16)_
