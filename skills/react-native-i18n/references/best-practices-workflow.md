---
name: react-native-i18n-workflow
description: Translation workflow for React Native — key naming, string extraction with i18next-cli, missing-key checks in CI, pseudo-localization, per-app language settings, localized app names and store metadata.
---

# i18n Workflow

## Writing translatable strings

| Do | Don't |
|----|-------|
| `t('checkout.confirm', { total })` → "Pay {{total}} now" | `t('pay') + ' ' + total + ' ' + t('now')` |
| One key per full message | Reuse a key because the English happens to match ("Open" door vs "Open" status) |
| Interpolate values and let translators place them | Assume English word order |
| `<Trans>` for links/bold inside a sentence | Split a sentence into separate `<Text>` keys |
| Leave room: German and Finnish run ~30–40% longer | Fixed-width buttons sized for English |

**Key naming:** feature-scoped, describing purpose, not English text — `profile.edit.saveButton`, `errors.network.offline`. Add translator context (where the string appears, max length) as key descriptions in your TMS.

## Extraction and checks

Generate and sync keys from source with `i18next-cli`:

```bash
npm install -D i18next-cli
npx i18next-cli init       # writes i18next.config.ts
npx i18next-cli extract    # scans t() / <Trans> and updates locale files
npx i18next-cli status     # translation coverage per locale
npx i18next-cli lint       # flags hard-coded strings in JSX
```

```ts
// i18next.config.ts
import { defineConfig } from 'i18next-cli';

export default defineConfig({
  locales: ['en', 'pt-BR', 'ar'],
  extract: {
    input: ['src/**/*.{ts,tsx}', 'app/**/*.{ts,tsx}'],
    output: 'src/i18n/locales/{{language}}/{{namespace}}.json',
  },
});
```

**CI gate:** run `npx i18next-cli extract --ci --dry-run` (non-zero exit when files would change) so new strings can't be merged without keys, and fail the build when a shipped locale is missing keys. Typed keys (`CustomTypeOptions`) catch typos at compile time.

## Pseudo-localization

Before real translations exist, catch hard-coded strings, truncation and concatenation with a pseudo-locale (`en-XA` style): accents plus ~40% padding, e.g. "Šåṽé" → "[Šåṽé ~~~~]".

```ts
// dev only: generate a pseudo locale from en
const pseudo = (s: string) =>
  `[${s.replace(/[aeiou]/g, (c) => ({ a: 'å', e: 'é', i: 'î', o: 'ö', u: 'ü' })[c]!)}${'~'.repeat(Math.ceil(s.length * 0.4))}]`;
```

Anything that shows plain English while the pseudo locale is active is hard-coded. Pair with forced RTL for layout checks.

## Per-app language (OS settings)

Android 13+ and iOS let users pick a language per app in system settings. Declare the locales you support:

```json
{
  "expo": {
    "plugins": [
      ["expo-localization", { "supportedLocales": ["en", "pt-BR", "ar"] }]
    ]
  }
}
```

Use `{ "ios": [...], "android": [...] }` if the lists differ. On Android this also limits bundled resource configurations. `allowDynamicLocaleChangesAndroid` keeps the activity from restarting when the language changes.

Open the OS screen from your settings page instead of building a duplicate picker:

```ts
import { Linking } from 'react-native';
import { openAppLanguageSettings } from 'react-native-localize'; // bare

await Linking.openSettings();          // Expo / iOS
await openAppLanguageSettings();       // Android 13+ app language screen
```

Read the result on next foreground with `getLocales()` / `useLocales()`.

Bare apps: add `CFBundleLocalizations` to `Info.plist` and `android:localeConfig="@xml/locales_config"` with a `locales_config.xml`.

## Localized app name and permission strings

```json
// app.json
{ "expo": { "locales": { "pt-BR": "./locales/app/pt-BR.json", "ar": "./locales/app/ar.json" } } }
```

```json
// locales/app/pt-BR.json
{
  "ios": {
    "CFBundleDisplayName": "Minha Loja",
    "NSCameraUsageDescription": "Usamos a câmera para escanear códigos."
  },
  "android": { "app_name": "Minha Loja" }
}
```

Bare apps use `InfoPlist.strings` per `.lproj` and `res/values-<qualifier>/strings.xml`.

## Store and release

- Localize store listings, screenshots and "What's New" text for each shipped locale (App Store Connect, Play Console, or `eas metadata`).
- Ship a locale only when it's complete; partial translations with English fallbacks look broken.
- Track per-locale crash and conversion metrics — formatting bugs often show up as locale-specific crashes.

## Translation management

Keep JSON in git as the source of truth and sync with a TMS (Crowdin, Lokalise, Phrase, Tolgee, Weblate) through its CLI or GitHub integration. Over-the-air translation updates (via the TMS SDK or i18next backend fetch) avoid app releases but need caching, versioning and a bundled fallback.

## Key Points

- Full-sentence keys with interpolation; no concatenation; budget for longer text.
- Extract keys with `i18next-cli`, gate CI on missing keys, use typed keys.
- Pseudo-localize and force RTL during development.
- Declare `supportedLocales` for per-app language; localize app name and permission strings; ship complete locales only.
