---
name: nextjs-16-patterns
description: Use when writing or reviewing any Next.js 16+ code. Audits for breaking changes vs. older Next.js training data — proxy.ts (not middleware.ts), 'use cache' + cacheLife (not export const dynamic/revalidate/fetchCache), awaited params/searchParams (Promises), updateTag (not revalidateTag) after mutations, and load-bearing unstable_* names.
---

# Next.js 16 Patterns

Next.js 16 has breaking changes vs. training-data Next.js. Verify the relevant docs in `node_modules/next/dist/docs/01-app/` before applying patterns — do not rely on memory.

## Five non-negotiable patterns

### 1. `proxy.ts`, not `middleware.ts`
- Edge-runtime entry is `proxy.ts` at the project root.
- The proxy auth check is **optimistic** — it cannot be the final authority. Every Server Action / DAL function must still re-verify the session.
- Never `fetch()` data in `proxy.ts`; cache options have no effect there.

### 2. `'use cache'` + `cacheLife()`, not `export const dynamic|revalidate|fetchCache`
```ts
// WRONG
export const dynamic = 'force-static';
export const revalidate = 3600;
export const fetchCache = 'force-cache';

// RIGHT
'use cache';
import { cacheLife } from 'next/cache';
cacheLife('hours');
```

### 3. `params` and `searchParams` are Promises — always `await`
```ts
// WRONG
export default function Page({ params }) {
  const { id } = params;
}

// RIGHT
export default async function Page({ params }) {
  const { id } = await params;
}
```

### 4. `updateTag()` after mutations, not `revalidateTag()`
- `updateTag()` gives read-your-own-writes consistency inside the same request.
- `revalidateTag()` is for out-of-band invalidation (cron, webhook), not mutation flows.

### 5. `unstable_*` and `useLinkStatus` names are load-bearing
- Never rename `unstable_instant`, `unstable_catchError`, `unstable_retry`, `useLinkStatus`.
- The prefix signals the API contract is experimental; renaming hides that.

## How to use this skill

Given code, identify violations by rule number, name the file:line, and supply the corrected snippet. If unsure of a behavior, cite `node_modules/next/dist/docs/01-app/...` rather than guessing.
