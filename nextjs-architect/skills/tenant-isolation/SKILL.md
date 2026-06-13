---
name: tenant-isolation
description: Use when writing or reviewing any cached query in a multi-tenant SaaS. Enforces tenant isolation in cache keys — every 'use cache' function that depends on tenant data must take tenantId as an argument and tag with cacheTag('<entity>:<tenantId>'). Forgetting this leaks data between tenants — treat as P0.
---

# Tenant Isolation in Cache Keys

In a multi-tenant SaaS, cache keys that omit the tenant scope **leak data across tenants**. This is a P0 security invariant. Every cached query that depends on tenant context must encode the `tenantId` in both its arguments and its cache tag.

## The invariant

For every `'use cache'` query that depends on tenant:

1. `tenantId` is an **argument** (so the cache key includes it).
2. `cacheTag('<entity>:<tenantId>')` is called inside the function (so invalidation is tenant-scoped).
3. The underlying DB query filters by `tenantId` in its WHERE clause (defense in depth).

## Correct pattern

```ts
import { cacheTag, cacheLife } from 'next/cache';

export async function getBookings(tenantId: string) {
  'use cache';
  cacheLife('minutes');
  cacheTag(`bookings:${tenantId}`);
  return db.select().from(bookings).where(eq(bookings.tenantId, tenantId));
}
```

## Anti-patterns to flag

```ts
// MISSING tenantId argument → cache key collides across tenants
async function getBookings() {
  'use cache';
  cacheTag('bookings'); // BAD — global tag, leaks across tenants
}

// tenantId pulled from session inside the cached function
async function getBookings() {
  'use cache';
  const { tenantId } = await getSession(); // BAD — session is not part of the cache key
}

// Tag without tenantId
async function getBookings(tenantId: string) {
  'use cache';
  cacheTag('bookings'); // BAD — invalidation is too broad / wrong shape
}
```

## Mutation invalidation pattern

After mutating tenant-scoped data inside a Server Action:

```ts
import { updateTag } from 'next/cache';
updateTag(`bookings:${session.tenantId}`); // scoped invalidation, read-your-own-writes
```

## How to use this skill

Grep for `'use cache'` across the codebase. For each match, verify:
- The function takes `tenantId` (or equivalent tenant key) as a parameter.
- `cacheTag` interpolates that tenant key.
- The DB query filters by tenant.

For each Server Action that mutates tenant data, verify `updateTag('<entity>:<tenantId>')` is called.
