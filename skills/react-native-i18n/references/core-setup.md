---
name: react-native-i18n-setup
description: Setting up i18n in React Native — device locale detection with expo-localization or react-native-localize, i18next 26 + react-i18next 17 init, resource layout, typed keys, language switching and persistence.
---

# i18n Setup

## Install

```bash
# Expo
npx expo install expo-localization
npm install i18next react-i18next

# Bare React Native
npm install react-native-localize i18next react-i18next
cd ios && pod install
```

i18next 24+ uses the v4 JSON format (`_one`, `_other`, … suffixes) and relies on `Intl.PluralRules`, which Hermes provides.

## Resource layout

```
src/i18n/
├── index.ts
└── locales/
    ├── en/
    │   ├── common.json
    │   └── checkout.json
    ├── pt-BR/
    │   ├── common.json
    │   └── checkout.json
    └── ar/
        └── ...
```

```json
// locales/en/common.json
{
  "greeting": "Hello, {{name}}!",
  "cart": {
    "items_one": "{{count}} item",
    "items_other": "{{count}} items"
  }
}
```

Split namespaces by feature so translators and code owners can work independently, and so large apps can lazy-load.

## Detect the device locale

```ts
// Expo
import { getLocales } from 'expo-localization';

const SUPPORTED = ['en', 'pt-BR', 'ar'] as const;

export function detectLanguage(): (typeof SUPPORTED)[number] {
  for (const { languageTag, languageCode } of getLocales()) {
    if (SUPPORTED.includes(languageTag as any)) return languageTag as any;
    const byLang = SUPPORTED.find((s) => s.split('-')[0] === languageCode);
    if (byLang) return byLang;
  }
  return 'en';
}
```

```ts
// Bare: react-native-localize does the matching for you
import { findBestLanguageTag } from 'react-native-localize';

const best = findBestLanguageTag(['en', 'pt-BR', 'ar']);
const lng = best?.languageTag ?? 'en';
```

`getLocales()` returns the user's preferred list in order, each with `languageTag`, `languageCode`, `regionCode`, `textDirection`, `currencyCode`, and separators. Use the region for formatting even when the UI language falls back (e.g. `en` UI with `pt-BR` number format).

## Initialize i18next

```ts
// src/i18n/index.ts
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import en from './locales/en/common.json';
import ptBR from './locales/pt-BR/common.json';
import ar from './locales/ar/common.json';
import { detectLanguage } from './detect';
import { storage } from '../storage';

export const resources = {
  en: { common: en },
  'pt-BR': { common: ptBR },
  ar: { common: ar },
} as const;

i18n.use(initReactI18next).init({
  resources,
  lng: storage.getString('language') ?? detectLanguage(), // user override first
  fallbackLng: 'en',
  supportedLngs: Object.keys(resources),
  defaultNS: 'common',
  ns: ['common'],
  interpolation: { escapeValue: false }, // React already escapes
  returnNull: false,
});

export default i18n;
```

Import `./i18n` once at the top of the root layout (`app/_layout.tsx`) or `index.js`, before any screen renders. With bundled `resources`, init is synchronous enough for the first render.

## Use in components

```tsx
import { useTranslation, Trans } from 'react-i18next';

export function CartHeader({ name, count }: { name: string; count: number }) {
  const { t } = useTranslation(); // or useTranslation('checkout')
  return (
    <>
      <Text>{t('greeting', { name })}</Text>
      <Text>{t('cart.items', { count })}</Text>
      <Trans i18nKey="terms" components={{ link: <Text style={styles.link} onPress={openTerms} /> }} />
    </>
  );
}
```

```json
{ "terms": "By continuing you accept the <link>terms</link>." }
```

Outside React (API errors, notifications): `import i18n from './i18n'; i18n.t('errors.network')`.

## Typed keys

```ts
// src/i18n/i18next.d.ts
import 'i18next';
import type { resources } from './index';

declare module 'i18next' {
  interface CustomTypeOptions {
    defaultNS: 'common';
    resources: (typeof resources)['en'];
  }
}
```

`t('cart.itms')` now fails type checking, and interpolation variables are checked against the source strings.

## Switching language at runtime

```ts
export async function setLanguage(lng: string) {
  await i18n.changeLanguage(lng);
  storage.set('language', lng);
  // Direction change? See features-rtl.md — requires a reload.
}
```

React components using `useTranslation` re-render automatically. Also react to OS changes: `useLocales()` (expo-localization) re-renders when the device locale changes while the app is running; only apply it if the user hasn't chosen a language in-app.

## Lazy-loading namespaces

For large apps, load namespaces on demand with a backend plugin (e.g. `i18next-resources-to-backend`):

```ts
import resourcesToBackend from 'i18next-resources-to-backend';

// Metro only bundles static import() paths, so list each loader explicitly
const loaders: Record<string, () => Promise<{ default: object }>> = {
  'en/checkout': () => import('./locales/en/checkout.json'),
  'pt-BR/checkout': () => import('./locales/pt-BR/checkout.json'),
};

i18n
  .use(resourcesToBackend((lng: string, ns: string) => loaders[`${lng}/${ns}`]?.()))
  .use(initReactI18next)
  .init({ fallbackLng: 'en', ns: ['common'], defaultNS: 'common' });
```

## Key Points

- Detect from `getLocales()` / `findBestLanguageTag()`, then let a persisted user choice override it.
- Bundle resources for instant first render; namespace by feature.
- `interpolation.escapeValue: false` in React Native; use `<Trans>` for embedded components.
- Augment `CustomTypeOptions` for typed keys.
