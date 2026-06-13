---
name: dal-discipline
description: Use when writing or reviewing authenticated reads, Server Actions, or any code under app/ that touches user data. Enforces the Data Access Layer (DAL) pattern — pages/components never query the DB directly; every authenticated read goes through src/lib/auth/dal.ts; every Server Action starts with verifySession().
---

# DAL Discipline

The **Data Access Layer** is the single, mandatory entry point for authenticated database reads. Pages and components must never query the DB directly. Server Actions must always begin with a session check.

## Hard rules

### 1. The DAL is the only read path for authenticated data
- All authenticated DB reads live in `src/lib/auth/dal.ts` (or a feature DAL it composes).
- Pages, layouts, route handlers, and components import DAL functions — never raw DB clients.
- A DAL function is responsible for: session verification → permission check → tenant scope → query.

### 2. Every Server Action's first line is `verifySession()`
```ts
'use server';
export async function deleteX(formData: FormData) {
  const session = await verifySession(); // FIRST LINE — non-negotiable
  // ... validate input ...
  // ... mutate ...
  updateTag(`x:${session.tenantId}`);
}
```

### 3. The proxy auth check is optimistic — never the final authority
- `proxy.ts` may short-circuit unauthenticated routes for UX, but it cannot be trusted by code that mutates or reads sensitive data.
- Always re-verify in the DAL / Server Action.

### 4. DAL functions encode the trust boundary
- Inputs come from untrusted callers; outputs go to trusted callers.
- Validate IDs and inputs at the DAL layer with a schema (Zod or equivalent) — not in the page.

## How to use this skill

When reviewing code:
1. Open every `'use server'` file. Confirm the first executable line is a session check.
2. Open every page / component that reads user data. Confirm it calls a DAL function, not a DB client.
3. Open every DAL function. Confirm it verifies session, scopes by tenant, and validates inputs.

Report any violation with file:line and the smallest correct refactor.
