---
name: react-native-i18n
description: Internationalization for React Native and Expo — device locale detection with expo-localization or react-native-localize, i18next/react-i18next setup, plurals and Intl formatting, RTL layouts, per-app language, and translation workflow.
metadata:
  author: maikotrindade
  version: "2026.10.0"
---

# React Native Internationalization

> Targets React Native 0.8x on Hermes, Expo SDK 54–58, i18next 26 and react-i18next 17. The same patterns apply to other libraries (Lingui, FormatJS); only the translation API changes.

## Core References

| Topic | Description | Reference |
|-------|-------------|-----------|
| Setup | Locale detection, i18next + react-i18next init, resource layout, typed keys, language switching and persistence | [core-setup](references/core-setup.md) |

## Features

| Topic | Description | Reference |
|-------|-------------|-----------|
| Plurals & formatting | Plural categories, interpolation, `Intl` dates/numbers/currency/relative time on Hermes, polyfills | [features-formatting](references/features-formatting.md) |
| RTL | `I18nManager`, `start`/`end` styles, icon mirroring, Expo `supportsRTL`, reload on direction change | [features-rtl](references/features-rtl.md) |

## Best Practices

| Topic | Description | Reference |
|-------|-------------|-----------|
| Workflow | Key naming, extraction, missing-key checks, pseudo-localization, per-app language settings, store metadata | [best-practices-workflow](references/best-practices-workflow.md) |

## Key Recommendations

- **Read locales from the OS** — `getLocales()` (expo-localization) or `getLocales()` / `findBestLanguageTag()` (react-native-localize); never parse `navigator.language`
- **Format with `Intl`, not string concatenation** — dates, numbers, currency and plurals depend on locale rules, not just language
- **One key per message, full sentences** — no concatenated fragments; give translators context
- **Use `start`/`end`, not `left`/`right`** — layouts mirror automatically when RTL is on
- **Direction changes need a reload** — `I18nManager.forceRTL` takes effect only after restarting the JS bundle
- **Declare supported locales natively** — `supportedLocales` in the expo-localization plugin enables per-app language on Android 13+ and iOS
