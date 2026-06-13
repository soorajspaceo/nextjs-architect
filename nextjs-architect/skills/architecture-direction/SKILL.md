---
name: architecture-direction
description: Use when adding new files, moving code, or reviewing imports. Enforces the one-way dependency direction app → features → shared → ui. Cross-feature imports are banned (features/x cannot import features/y); shared cannot import features; ui (shadcn primitives) cannot import from anywhere above.
---

# Architecture Direction

The codebase is layered. Dependencies flow **one way only**:

```
app/  →  features/<x>/  →  components/shared/ + lib/  →  components/ui/
```

Reverse dependencies and lateral feature-to-feature imports are forbidden.

## The four layers

| Layer | What lives here | Can import from |
|---|---|---|
| `app/` | Routes, pages, layouts, route handlers, Server Actions | features, shared, lib, ui |
| `features/<x>/` | One feature's UI + actions + DAL + hooks + types | shared, lib, ui (not other features) |
| `components/shared/` + `lib/` | Cross-feature primitives (e.g. `<EmptyState>`, `formatMoney`) | ui, other shared/lib |
| `components/ui/` | shadcn primitives — Button, Input, Card | nothing above; only other ui |

## Hard rules

### 1. `features/<x>/` cannot import from `features/<y>/`
If two features need the same thing, **hoist it** to `components/shared/` (UI) or `lib/` (logic) in the same PR. Do not import sideways.

```ts
// WRONG (features/booking/list.tsx)
import { CustomerAvatar } from '@/features/customer/components/avatar';

// RIGHT — hoist CustomerAvatar to components/shared/ first, then both features import from there.
import { CustomerAvatar } from '@/components/shared/customer-avatar';
```

### 2. `components/shared/` cannot import from `features/*`
Shared is a leaf of the features layer — it has no knowledge of any specific feature.

### 3. `components/ui/` cannot import from anywhere above
`ui/` is shadcn primitives. To add domain behavior, **wrap** the primitive in `components/shared/` or `features/`. Do not edit `ui/` directly.

```ts
// WRONG — adding domain logic to ui/button.tsx
// (file: components/ui/button.tsx)
import { trackEvent } from '@/lib/analytics'; // analytics-aware Button — leaks domain into ui/

// RIGHT — wrap in shared
// (file: components/shared/analytics-button.tsx)
import { Button } from '@/components/ui/button';
import { trackEvent } from '@/lib/analytics';
export const AnalyticsButton = (props) => <Button onClick={() => { trackEvent(...); props.onClick?.(); }} />;
```

### 4. `app/` is the top — never imported by anything else
Pages and route handlers are entry points. Never import from `app/` into `features/`, `shared/`, `lib/`, or `ui/`.

## How to use this skill

Given a diff or a file:

1. List its imports.
2. Classify each by layer.
3. Flag any import that violates the direction above.

For each violation, propose the smallest fix — usually "hoist X to shared/" or "wrap primitive in shared/".
