# intersoccer-ci

Shared InterSoccer GitHub Actions: reusable PR CI for WordPress plugins and child theme.

## Overview

This repository contains **reusable GitHub Actions workflows** that product repositories call via `uses:`. These workflows provide consistent CI checks across all InterSoccer projects.

**Important:** CI results are advisory only — they are not configured as required status checks. Deploy and publish workflows remain in the individual product repositories.

## Available Workflows

### php-plugin-ci.yml

Reusable CI workflow for WordPress plugin repositories.

**Jobs:**
- **lint** — PHP 8.2 syntax check on plugin files (root `*.php`, `includes/`, `src/`, `classes/`, `elementor/`)
- **test** — Runs `composer test` if defined, otherwise `vendor/bin/phpunit`

**Requirements:**
- Repository must have a `composer.json` in the root
- Uses each repo's own `composer.json` / `phpunit.xml` configuration

### theme-smoke.yml

Lightweight smoke test for the child theme repository.

**Jobs:**
- **smoke** — PHP 8.2 syntax check on theme PHP files (`functions.php`, `page.php`, standard theme files, `templates/`, `template-parts/`, `woocommerce/`, `inc/`)

**Requirements:**
- No composer required

### publish-plugin.yml

Reusable workflow for publishing WordPress plugin ZIPs to [Underdog Unlimited](https://plugins.underdogunlimited.com).

**Jobs:**
- **publish** — Builds a distributable ZIP (excluding `.git`, `.github`, `tests`, `vendor`, dev artifacts) and POSTs it to the Underdog Unlimited publish API

**Inputs:**
| Input | Required | Description |
|-------|----------|-------------|
| `plugin_slug` | Yes | UU Admin slug / ZIP folder name (e.g. `player-management`) |
| `bootstrap_php` | Yes | Main plugin PHP filename at repo root (e.g. `player-management.php`) |
| `version` | Yes | Version to publish (e.g. `2.7.20` or `2.7.20-rc3`, without leading `v`) |
| `changelog` | No | Release notes / changelog text |
| `publish_url` | No | Publish base URL (default: `https://plugins.underdogunlimited.com`) |

**Secrets:**
| Secret | Required | Description |
|--------|----------|-------------|
| `UU_PUBLISH_TOKEN` | Yes | Publish token (`udpub_…`) |

**Requirements:**
- Bootstrap PHP file must exist at repo root
- Caller handles trigger (`on: release`, `workflow_dispatch`) and computes version

## Usage

### Plugin Repository Caller Example

Create `.github/workflows/ci.yml` in your plugin repository:

```yaml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  ci:
    uses: legit-ninja/intersoccer-ci/.github/workflows/php-plugin-ci.yml@main
```

### Theme Repository Caller Example

Create `.github/workflows/ci.yml` in your theme repository:

```yaml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  smoke:
    uses: legit-ninja/intersoccer-ci/.github/workflows/theme-smoke.yml@main
```

### Publish Plugin Caller Example

Create `.github/workflows/publish.yml` in your plugin repository:

```yaml
name: Publish to Underdog

on:
  release:
    types: [published]
  workflow_dispatch:
    inputs:
      version:
        description: "Version to publish (e.g. 2.7.20 or 2.7.20-rc3)"
        required: true
        type: string

jobs:
  publish:
    uses: legit-ninja/intersoccer-ci/.github/workflows/publish-plugin.yml@main
    with:
      plugin_slug: player-management
      bootstrap_php: player-management.php
      version: ${{ github.event_name == 'workflow_dispatch' && github.event.inputs.version || github.ref_name }}
      changelog: ${{ github.event.release.body || '' }}
    secrets:
      UU_PUBLISH_TOKEN: ${{ secrets.UU_PUBLISH_TOKEN }}
```

**Notes on the caller:**
- The caller computes `version` from the release tag (`github.ref_name`) or manual input
- `changelog` is pulled from the GitHub release body when available
- Add the `UU_PUBLISH_TOKEN` secret to your repository (or use GitHub Environments for staging/production separation)
- The reusable workflow strips any leading `v` from the version automatically

## Notes

- **PHP version:** All workflows use PHP 8.2
- **No required checks:** These workflows do not enforce branch protection rules. Configure rulesets in each product repository if blocking merges is desired.
- **Extensible:** Future versions may add inputs for PHP version selection, PHPCS, PHPStan, etc.
