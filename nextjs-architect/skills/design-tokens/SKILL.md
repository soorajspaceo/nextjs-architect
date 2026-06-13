---
name: design-tokens
description: Use when writing or reviewing any UI / Tailwind / CSS code. Enforces design-token discipline — no raw hex or oklch in JSX (CSS variables only), Tailwind logical properties only (ms-/me-/ps-/pe-, never ml-/mr-/pl-/pr-), rounded-full only on avatars and icon-only utility buttons.
---

# Design Token Discipline

Design tokens are the contract between design and code. Hard-coding values in JSX breaks theming, dark mode, and RTL. Three rules below cover the most-violated cases.

## Rule 1 — Colors come from CSS variables, never raw values

```tsx
// WRONG
<div className="bg-[#0a0a0a] text-[oklch(0.5 0 0)]">

// RIGHT — semantic CSS variables defined in app/globals.css @theme
<div className="bg-background text-foreground">
```

The set of variables (`--background`, `--foreground`, `--primary`, `--accent`, etc.) is defined by the design system. If a needed token does not exist, add it to `@theme` rather than inlining a raw value.

## Rule 2 — Tailwind **logical** properties only

Directional classes break RTL layouts. The codebase supports `en` (LTR) and `ar` (RTL).

| Banned | Use instead |
|---|---|
| `ml-4` / `mr-4` | `ms-4` / `me-4` |
| `pl-2` / `pr-2` | `ps-2` / `pe-2` |
| `left-0` / `right-0` | `start-0` / `end-0` |
| `border-l` / `border-r` | `border-s` / `border-e` |
| `text-left` / `text-right` | `text-start` / `text-end` |

## Rule 3 — `rounded-full` only on avatars and icon-only utility buttons

```tsx
// WRONG — pill-shaped CTA button
<Button className="rounded-full">Book now</Button>

// RIGHT — use the design system's button radius
<Button>Book now</Button>

// OK — avatar
<Avatar className="rounded-full" />

// OK — icon-only utility (close, settings, etc.)
<IconButton className="rounded-full" aria-label="Close" />
```

Use `rounded-md`, `rounded-lg`, etc. (defined in `@theme`) for everything else.

## How to use this skill

Scan JSX / Tailwind classes for:
1. `bg-[`, `text-[`, `border-[`, or any `#`/`oklch(` inside `className`.
2. `m[lr]-`, `p[lr]-`, `border-[lr]`, `text-(left|right)`, `(left|right)-` positioning.
3. `rounded-full` on elements other than avatars and aria-labeled icon-only buttons.

For each hit, report file:line and the fix.
