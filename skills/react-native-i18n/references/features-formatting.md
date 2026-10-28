---
name: react-native-i18n-formatting
description: Plurals, interpolation and locale-aware formatting in React Native — CLDR plural categories in i18next, Intl.NumberFormat/DateTimeFormat/RelativeTimeFormat on Hermes, i18next formatters, time zones, and polyfills.
---

# Plurals & Formatting

## Plurals

Languages have up to six CLDR plural categories: `zero`, `one`, `two`, `few`, `many`, `other`. English uses two; Arabic uses all six; Japanese uses only `other`. Never write `count === 1 ? 'item' : 'items'` in code.

```json
// en
{ "messages_one": "{{count}} new message", "messages_other": "{{count}} new messages" }

// ar
{
  "messages_zero": "لا رسائل جديدة",
  "messages_one": "رسالة جديدة واحدة",
  "messages_two": "رسالتان جديدتان",
  "messages_few": "{{count}} رسائل جديدة",
  "messages_many": "{{count}} رسالة جديدة",
  "messages_other": "{{count}} رسالة جديدة"
}
```

```ts
t('messages', { count: 3 });
```

- The variable must be named `count` for i18next to pick the plural form.
- Ordinals ("1st", "2nd"): pass `ordinal: true` and use `_ordinal_one`, `_ordinal_two`, `_ordinal_few`, `_ordinal_other` keys.
- Check categories for a locale: `new Intl.PluralRules('pl').resolvedOptions().pluralCategories`.

## Context (gender, variants)

```json
{ "invite": "They invited you", "invite_male": "He invited you", "invite_female": "She invited you" }
```

```ts
t('invite', { context: user.gender }); // falls back to "invite" when context is unknown
```

## Numbers, currency, units

```ts
const locale = i18n.language; // or getLocales()[0].languageTag for region-specific formatting

new Intl.NumberFormat(locale).format(1234567.891);                          // 1,234,567.891 / 1.234.567,891
new Intl.NumberFormat(locale, { style: 'currency', currency: 'BRL' }).format(42); // R$ 42,00
new Intl.NumberFormat(locale, { style: 'percent' }).format(0.25);           // 25%
new Intl.NumberFormat(locale, { notation: 'compact' }).format(12500);        // 12.5K / 12,5 mil
new Intl.NumberFormat(locale, { style: 'unit', unit: 'kilometer' }).format(5);
```

The currency comes from the data (what the price is in), not from the user's locale. Use `getLocales()[0].currencyCode` only as a default for user input.

## Dates and times

```ts
const df = new Intl.DateTimeFormat(locale, { dateStyle: 'medium', timeStyle: 'short' });
df.format(new Date(order.createdAt));

new Intl.DateTimeFormat(locale, { weekday: 'long', day: 'numeric', month: 'long' }).format(date);
new Intl.DateTimeFormat(locale, { timeZone: 'America/Sao_Paulo', hour: 'numeric', minute: '2-digit' }).format(date);
```

- Store and transmit UTC (ISO 8601); format at the edge with the user's time zone (`getCalendars()[0].timeZone`).
- Respect the device 12/24h setting: `getCalendars()[0].uses24hourClock` (Expo) or `uses24HourClock()` (react-native-localize) → pass `hourCycle: 'h23'` or `'h12'`.
- Create formatters once (module scope or `useMemo`) — constructing `Intl` objects in list rows is slow.

## Relative time and lists

```ts
new Intl.RelativeTimeFormat(locale, { numeric: 'auto' }).format(-1, 'day'); // "yesterday" / "ontem"
new Intl.ListFormat(locale, { type: 'conjunction' }).format(['Ana', 'Bo', 'Cy']); // "Ana, Bo, and Cy"
```

## Hermes Intl support and polyfills

Hermes implements the core `Intl` APIs (`DateTimeFormat`, `NumberFormat`, `PluralRules`, `Collator`, `getCanonicalLocales`) backed by the platform's ICU/Foundation. Newer APIs and options may be missing or incomplete depending on RN version and platform — feature-detect before relying on them:

```ts
const hasRelative = typeof Intl.RelativeTimeFormat === 'function';
const hasList = typeof Intl.ListFormat === 'function';
```

If missing, polyfill with FormatJS, loading only the locales you ship:

```ts
// index.js — before i18n init
import '@formatjs/intl-getcanonicallocales/polyfill-force';
import '@formatjs/intl-locale/polyfill-force';
import '@formatjs/intl-relativetimeformat/polyfill';
import '@formatjs/intl-relativetimeformat/locale-data/en';
import '@formatjs/intl-relativetimeformat/locale-data/pt';
```

Order matters (`getcanonicallocales` → `locale` → `pluralrules` → `numberformat` → others), and each locale-data import adds bundle size.

## i18next formatters

i18next has built-in `Intl` formatters usable inside strings:

```json
{
  "total": "Total: {{amount, currency(USD)}}",
  "updated": "Updated {{date, datetime(dateStyle: short)}}",
  "progress": "{{value, number(style: percent)}} done"
}
```

```ts
t('total', { amount: 19.9 });
t('updated', { date: new Date() });
```

This keeps word order with the translator: some languages put the number after the noun.

## Collation and search

```ts
const collator = new Intl.Collator(locale, { sensitivity: 'base' });
names.sort(collator.compare);                 // locale-aware sort (å, ö, ñ…)
collator.compare('resume', 'résumé') === 0;   // accent-insensitive match
```

## Key Points

- Plurals by CLDR category with `count`; never branch on `=== 1` in code.
- `Intl.NumberFormat` / `DateTimeFormat` for every user-visible number and date; reuse formatter instances.
- Store UTC, format with the device time zone and hour cycle.
- Feature-detect newer `Intl` APIs on Hermes; polyfill with FormatJS only for shipped locales.
