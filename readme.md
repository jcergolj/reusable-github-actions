# Reusable GitHub Actions

Reusable GitHub Actions for Laravel applications.

## Reusable workflows

Call workflows from an application workflow with `uses: jcergolj/reusable-github-actions/.github/workflows/<name>.yml@master`.

| Workflow | Purpose | Inputs / secrets |
| --- | --- | --- |
| `tests.yml` | Runs Laravel tests on PHP 8.5 by default | `install-tailwind-css`, `php-versions` inputs |
| `larastan.yml` | Runs `vendor/bin/phpstan analyse --memory-limit=2G` on PHP 8.5 | None |
| `pint.yml` | Checks Laravel Pint formatting | None |
| `rector.yml` | Checks Rector changes without modifying files | None |
| `code-formatter.yml` | Runs Prettier, Rector, and Laravel Pint | `skip-npm`, `pint-blade` inputs; `PAT_TOKEN` secret for commits |
| `phpmd.yml` | Runs `composer phpmd` on one PHP version | `php-version` input |
| `envy.yml` | Runs Envy sync and prune checks | None |
| `gitleaks.yml` | Scans the full Git history for secrets | `PAT_TOKEN` secret |
| `trufflehog-scan.yml` | Scans the repository for secrets | `PAT_TOKEN` secret |

## Composite actions

Use these inside a normal or reusable workflow after checkout when appropriate.

| Action | Purpose | Inputs |
| --- | --- | --- |
| `php-setup` | Checks out the repository and installs PHP | `php-version` required |
| `composer-setup` | Caches and installs Composer dependencies | None |
| `npm-setup` | Installs NPM dependencies and runs Prettier | `skip-npm` optional |
| `rector` | Caches Rector data and runs Rector | `skip-rector` optional |

## Standard Laravel quality workflow

```yaml
name: CI

on:
  push:
    branches: [master]
  pull_request:

jobs:
  tests:
    uses: jcergolj/reusable-github-actions/.github/workflows/tests.yml@master
    with:
      php-versions: '["8.4", "8.5"]'

  larastan:
    uses: jcergolj/reusable-github-actions/.github/workflows/larastan.yml@master

  formatter:
    uses: jcergolj/reusable-github-actions/.github/workflows/code-formatter.yml@master
    secrets: inherit

  phpmd:
    uses: jcergolj/reusable-github-actions/.github/workflows/phpmd.yml@master
```

## PHPMD setup

The `phpmd.yml` workflow expects the application repository to define its own `composer phpmd` command and local PHPMD ruleset.

Example `composer.json` script:

```json
{
  "scripts": {
    "phpmd": "./vendor/bin/phpmd analyze app --format text --ruleset phpmd.xml"
  }
}
```

Minimal `phpmd.xml` starter:

```xml
<?xml version="1.0"?>
<ruleset name="Minimal PHPMD">
    <description>Complexity and size guardrails for app code.</description>

    <exclude-pattern>*/vendor/*</exclude-pattern>
    <exclude-pattern>*/bootstrap/cache/*</exclude-pattern>

    <rule ref="rulesets/codesize.xml/CyclomaticComplexity">
        <properties>
            <property name="reportLevel" value="10" />
        </properties>
    </rule>

    <rule ref="rulesets/codesize.xml/NPathComplexity">
        <properties>
            <property name="minimum" value="200" />
        </properties>
    </rule>

    <rule ref="rulesets/codesize.xml/ExcessiveMethodLength">
        <properties>
            <property name="minimum" value="75" />
        </properties>
    </rule>

    <rule ref="rulesets/codesize.xml/ExcessiveClassLength">
        <properties>
            <property name="minimum" value="500" />
        </properties>
    </rule>

    <rule ref="rulesets/codesize.xml/ExcessiveParameterList">
        <properties>
            <property name="minimum" value="6" />
        </properties>
    </rule>
</ruleset>
```

## Security notes

- Pin workflow references to a release tag or commit SHA for production use instead of `@master`.
- Store `PAT_TOKEN` as a GitHub secret.
