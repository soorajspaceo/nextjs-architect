# nextjs-architect

A [Claude Code](https://claude.com/claude-code) plugin marketplace with architectural guardrails for modern Next.js multi-tenant SaaS projects.

Install it once, and Claude will help you audit and enforce six common architectural patterns in any Next.js 16+ project — DAL discipline, tenant isolation, design tokens, i18n, dependency direction, and Next.js 16 breaking changes.

---

## What's inside

This marketplace ships one plugin, `nextjs-architect`, containing **six on-demand skills**:

| Skill | What it audits |
|---|---|
| `nextjs-16-patterns` | `proxy.ts` (not `middleware.ts`), `'use cache'` + `cacheLife()`, awaited `params`/`searchParams`, `updateTag()` (not `revalidateTag()`), load-bearing `unstable_*` names |
| `dal-discipline` | Every authenticated DB read goes through the DAL; every Server Action starts with `verifySession()` |
| `tenant-isolation` | Multi-tenant cache keys include `tenantId` in both function args and `cacheTag('<entity>:<tenantId>')` |
| `design-tokens` | CSS variables for colors (no raw hex/oklch); Tailwind logical props only (`ms-*`/`me-*` not `ml-*`/`mr-*`); `rounded-full` discipline |
| `i18n-discipline` | `next-intl` `t()` for all user-visible strings; RTL-safe layouts; locale-aware date/number/currency formatters |
| `architecture-direction` | One-way dependency direction: `app → features → shared → ui` (no cross-feature imports) |

---

## Installation

In any Claude Code session, run:

```
/plugin marketplace add soorajspaceo/nextjs-architect
/plugin install nextjs-architect@local
/reload-plugins
```

When prompted, pick **"Install for you (user scope)"** so the plugin is available in every project, not just the current repo.

Verify with:

```
/plugin list
```

You should see `nextjs-architect@local` listed as **enabled**.

---

## Usage

After installation, the six skills are invokable in any project:

```
/nextjs-architect:nextjs-16-patterns
/nextjs-architect:dal-discipline
/nextjs-architect:tenant-isolation
/nextjs-architect:design-tokens
/nextjs-architect:i18n-discipline
/nextjs-architect:architecture-direction
```

Each skill can be invoked alone (Claude will ask what to audit) or with a target file/snippet:

```
/nextjs-architect:nextjs-16-patterns audit src/app/[locale]/dashboard/page.tsx
/nextjs-architect:tenant-isolation check src/features/bookings/queries.ts
/nextjs-architect:design-tokens scan src/components/ui/header.tsx
```

Claude reports any violations with `file:line` references and the smallest correct refactor.

---

## Updating to the latest version

When this marketplace gets new commits:

```
/plugin marketplace update local
/reload-plugins
```

---

## Iterating on a skill

To tweak a skill's rules:

```bash
git clone https://github.com/soorajspaceo/nextjs-architect.git
cd nextjs-architect
# Edit nextjs-architect/skills/<skill-name>/SKILL.md
```

Test your changes locally before pushing:

```bash
claude --plugin-dir ./nextjs-architect
```

Then in that session, run `/nextjs-architect:<skill-name>` to validate behavior. When happy:

```bash
git add .
git commit -m "improve: <skill-name> ..."
git push
```

Teammates pick up the change with `/plugin marketplace update local` + `/reload-plugins`.

---

## Repository structure

```
.
├── .claude-plugin/
│   └── marketplace.json           # marketplace manifest
├── nextjs-architect/              # the plugin itself
│   ├── .claude-plugin/
│   │   └── plugin.json            # plugin manifest
│   └── skills/
│       ├── nextjs-16-patterns/SKILL.md
│       ├── dal-discipline/SKILL.md
│       ├── tenant-isolation/SKILL.md
│       ├── design-tokens/SKILL.md
│       ├── i18n-discipline/SKILL.md
│       └── architecture-direction/SKILL.md
└── README.md
```

---

## Why this exists

Modern Next.js + multi-tenant SaaS has a handful of architectural patterns that are easy to forget and expensive to get wrong:

- Cache keys without `tenantId` leak data across tenants (P0 security).
- Server Actions without `verifySession()` skip authentication entirely.
- `revalidateTag()` after a mutation breaks read-your-own-writes; `updateTag()` is the fix.
- `ml-*`/`mr-*` Tailwind classes break RTL layouts in Arabic, Hebrew, Persian.
- Raw hex colors in JSX break theming and dark mode.
- Cross-feature imports (`features/booking` → `features/customer`) make the codebase impossible to refactor.

These rules are easy to enforce in code review — and easy to forget when shipping fast. `nextjs-architect` lets you ask Claude to audit a file or snippet against any of these rules, on demand.
