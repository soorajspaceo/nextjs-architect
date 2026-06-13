---
name: i18n-discipline
description: Use when writing or reviewing JSX, copy, or anything user-visible. Enforces i18n discipline — no hardcoded user-visible strings in JSX (always go through next-intl t()), RTL-safe layout (uses logical CSS / Tailwind properties), and locale-aware formatters for dates/numbers/currency.
---

# i18n Discipline

Every user-visible string must route through `next-intl`. Layouts must work in both LTR and RTL (the project ships `en` and `ar`). Dates, numbers, and currency must use locale-aware formatters.

## Rule 1 — No hardcoded user-visible strings in JSX

```tsx
// WRONG
<Button>Book now</Button>
<p>Welcome back</p>
<img alt="Car photo" />

// RIGHT
const t = useTranslations('home');
<Button>{t('cta.bookNow')}</Button>
<p>{t('welcomeBack')}</p>
<img alt={t('carPhotoAlt')} />
```

Exceptions (no `t()` needed):
- Brand names, product names, code identifiers.
- Strings inside `console.log` / dev-only diagnostics.
- Strings inside test files.

## Rule 2 — `aria-label`, `placeholder`, `title`, `alt` are also user-visible

These attributes show to users (screen readers, hover, image-fallback). They are **not** exempt from translation.

```tsx
// WRONG
<input placeholder="Search bookings" aria-label="Search" />

// RIGHT
<input placeholder={t('search.placeholder')} aria-label={t('search.aria')} />
```

## Rule 3 — Layouts must be RTL-safe

See the `design-tokens` skill — use Tailwind logical properties (`ms-*`, `me-*`, `ps-*`, `pe-*`, `start-*`, `end-*`). Test the page in `ar` after any layout change.

## Rule 4 — Format dates, numbers, currency with locale-aware formatters

```tsx
// WRONG
<span>{date.toLocaleDateString('en-US')}</span>
<span>{`$${price}`}</span>

// RIGHT
const format = useFormatter();
<span>{format.dateTime(date, { dateStyle: 'medium' })}</span>
<span>{format.number(price, { style: 'currency', currency })}</span>
```

## How to use this skill

Scan JSX for:
1. String literals inside element children: `>Some text<` patterns where "Some text" is not a brand/code identifier.
2. String literals in `aria-label`, `placeholder`, `title`, `alt` attributes.
3. `toLocaleDateString` / `toLocaleString` / `${'$'}` template patterns for money.

For each, report file:line and the smallest correct refactor — including the suggested translation key.
