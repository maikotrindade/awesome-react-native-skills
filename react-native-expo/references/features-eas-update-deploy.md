---
name: react-native-expo-eas-update-deploy
description: EAS Update (OTA), EAS Hosting, and CI/CD YAML workflows — channels, rollouts, PR previews, production releases.
---

# EAS Update, Deploy & CI/CD

**EAS Update** pushes JavaScript-only over-the-air (OTA) updates to users without requiring a new native build. **EAS Hosting** deploys web apps and API routes. **EAS Workflows** automates builds, updates, and deployments via YAML.

## EAS Update (OTA)

OTA updates deliver new JavaScript bundles to users who already have the app installed. Only JS/asset changes are supported — native code changes require a new build.

```bash
# Publish an update to a channel
eas update --channel production --message "Fix cart total calculation"
eas update --channel preview --message "New onboarding flow"

# Target a specific branch
eas update --branch feature/new-ui --channel staging

# List recent updates
eas update:list
eas update:view <update-id>
```

**Channels vs branches:** A channel (e.g. `production`) points to a branch. Reassign channels to roll back or promote:

```bash
eas channel:edit production --branch previous-stable
```

## EAS Hosting (Web + API Routes)

EAS Hosting deploys Expo web builds and API routes on Cloudflare Workers.

```bash
# Production deploy
eas deploy

# Preview deploy (shareable URL, not promoted to production)
eas deploy --prod false

# List deployments
eas deploy:list
```

## EAS Workflows (CI/CD)

Workflow files live in `.eas/workflows/*.yml`. They automate builds, submissions, updates, and deploys triggered by git events or manual dispatch.

**Top-level structure:**

```yaml
name: Production Release
on:
  push:
    tags:
      - "v*"

jobs:
  build_and_submit:
    type: build
    params:
      platform: all
      profile: production
    build_and_submit: true
```

**Job types:** `build`, `submit`, `update`, `deploy`, `run` (custom shell commands).

**Triggers:**

```yaml
on:
  push:
    branches: [main]          # push to main
    tags: ["v*"]              # semver tags
  pull_request:
    branches: [main]          # PR against main
  schedule:
    - cron: "0 9 * * 1"       # every Monday at 9am UTC
  workflow_dispatch: {}       # manual trigger
```

**Expression syntax:** `${{ }}` with contexts:

```yaml
${{ github.ref }}             # git ref
${{ inputs.profile }}         # workflow input
${{ needs.build.outputs.buildId }}  # output from a previous job
${{ steps.my_step.outputs.result }} # output from a step
```

**Common workflows:**

*Web deploy on push to main:*

```yaml
name: Deploy Web
on:
  push:
    branches: [main]
jobs:
  deploy:
    type: deploy
    params:
      prod: true
```

*PR preview (web):*

```yaml
name: PR Preview
on:
  pull_request:
    branches: [main]
jobs:
  preview:
    type: deploy
    params:
      prod: false
```

*PR preview (native OTA):*

```yaml
name: PR OTA Preview
on:
  pull_request:
    branches: [main]
jobs:
  update:
    type: update
    params:
      channel: pr-${{ github.event.pull_request.number }}
      message: "PR #${{ github.event.pull_request.number }} preview"
```

*Production build + submit on tag:*

```yaml
name: Production Release
on:
  push:
    tags: ["v[0-9]+.[0-9]+.[0-9]+"]
jobs:
  build_ios:
    type: build
    params:
      platform: ios
      profile: production
  build_android:
    type: build
    params:
      platform: android
      profile: production
  submit_ios:
    needs: [build_ios]
    type: submit
    params:
      platform: ios
      profile: production
      build_id: ${{ needs.build_ios.outputs.buildId }}
  submit_android:
    needs: [build_android]
    type: submit
    params:
      platform: android
      profile: production
      build_id: ${{ needs.build_android.outputs.buildId }}
```

**Secrets in workflows:** Reference secrets from EAS environment:

```yaml
jobs:
  my_job:
    type: run
    env:
      DATABASE_URL: ${{ eas.env.DATABASE_URL }}
    steps:
      - run: node scripts/migrate.js
```

**Validate a workflow file:**

```bash
# Fetch schema and validate locally
node .eas/scripts/validate.js .eas/workflows/production.yml
```

## Key Points

- OTA updates (`eas update`) are JS-only; any native code change needs `eas build`
- Channels are pointers to branches — reassign to roll back without a new build
- Workflow YAML lives in `.eas/workflows/`; use `workflow_dispatch` for manual triggers
- Store all secrets in EAS environment variables, not hardcoded in workflow files

<!--
Source references:
- https://github.com/expo/skills (plugins/expo/skills/expo-deployment/references/workflows.md)
- https://github.com/expo/skills (plugins/expo/skills/expo-cicd-workflows/SKILL.md)
- https://docs.expo.dev/eas-update/introduction/
-->
