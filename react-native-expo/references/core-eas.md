---
name: react-native-expo-eas
description: EAS (Expo Application Services) overview — Build, Update, Submit, Deploy, eas.json structure, and version management.
---

# EAS Overview

EAS (Expo Application Services) provides cloud infrastructure for building, updating, and distributing Expo and React Native apps: **EAS Build** (native builds in the cloud), **EAS Update** (OTA JavaScript updates), **EAS Submit** (app store submissions), and **EAS Hosting** (web + API route deployment).

## Setup

```bash
npm install -g eas-cli
eas login
eas init          # generates eas.json and links project to Expo account
```

## eas.json Structure

`eas.json` defines build profiles for each stage:

```json
{
  "cli": {
    "version": ">= 16.0.0",
    "appVersionSource": "remote"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "autoIncrement": true
    },
    "preview": {
      "distribution": "internal",
      "autoIncrement": true
    },
    "production": {
      "autoIncrement": true
    }
  },
  "submit": {
    "production": {
      "ios": {
        "appleId": "you@example.com",
        "ascAppId": "123456789"
      },
      "android": {
        "serviceAccountKeyPath": "./google-service-account.json",
        "track": "internal"
      }
    }
  }
}
```

**`appVersionSource: "remote"`** — EAS manages version numbers automatically. You can still bump manually when needed:
```bash
eas build:version:set --platform ios --version-code 42
```

## Build Commands

```bash
# Start a build
eas build -p ios --profile production
eas build -p android --profile preview
eas build -p all --profile production

# Build locally (generates .ipa / .apk without EAS cloud)
eas build -p ios --profile development --local

# Check build status
eas build:list
eas build:view <build-id>
```

## Update Commands (OTA)

```bash
# Push a JavaScript-only OTA update to a channel
eas update --channel production --message "Fix checkout bug"
eas update --channel preview --branch feature/new-ui

# List recent updates
eas update:list
```

## Submit Commands

```bash
# Submit latest build to app store
eas submit -p ios --latest
eas submit -p android --latest

# Submit specific build
eas submit -p ios --id <build-id>

# iOS shortcut: build + submit to TestFlight in one step
eas build -p ios --profile production --auto-submit
```

## Deploy Commands (Web + API Routes)

```bash
# Deploy to EAS Hosting
eas deploy                        # production
eas deploy --prod false           # preview (shareable URL)

# Check deployment
eas deploy:list
```

## Environment Variables / Secrets

```bash
# Create a secret (available in API routes as process.env.MY_SECRET)
eas env:create --name MY_SECRET --value "supersecret" --environment production

# Client-side vars: EXPO_PUBLIC_ prefix (inlined at build time, visible in bundle)
# Never use EXPO_PUBLIC_ for secrets
```

## Key Points

- Run `eas build` for full native builds; `eas update` for JavaScript-only OTA changes
- `appVersionSource: "remote"` eliminates manual version bumps in `app.json`
- Secrets belong in EAS environment variables, not `EXPO_PUBLIC_` vars or source code
- Use `--auto-submit` flag to combine build + App Store submission in a single command

<!--
Source references:
- https://github.com/expo/skills (plugins/expo/skills/expo-deployment/SKILL.md)
- https://docs.expo.dev/eas/
-->
