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

## Notes

- **PHP version:** All workflows use PHP 8.2
- **No required checks:** These workflows do not enforce branch protection rules. Configure rulesets in each product repository if blocking merges is desired.
- **No deploy/publish:** Deployment and publishing workflows belong in the product repositories, not here.
- **Extensible:** Future versions may add inputs for PHP version selection, PHPCS, PHPStan, etc.
