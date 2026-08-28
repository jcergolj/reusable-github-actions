# Reusable GitHub Actions

Reusable GitHub Actions for Laravel applications.

## Reusable workflows

Call workflows from an application workflow with `uses: jcergolj/reusable-github-actions/.github/workflows/<name>.yml@master`.

| Workflow | Purpose | Inputs / secrets |
| --- | --- | --- |
| `tests.yml` | Runs Laravel tests on PHP 8.4 and 8.5 | `install-tailwind-css` input |
| `larastan.yml` | Runs `vendor/bin/phpstan analyse --memory-limit=2G` on PHP 8.4 | None |
| `code-formatter.yml` | Runs Prettier, Rector, and Laravel Pint | `skip-npm`, `pint-blade` inputs; `PAT_TOKEN` secret for commits |
| `envy.yml` | Runs Envy sync and prune checks | None |
| `gitleaks.yml` | Scans the full Git history for secrets | `PAT_TOKEN` secret |
| `trufflehog-scan.yml` | Scans the repository for secrets | `PAT_TOKEN` secret |
| `deploy.yml` | Checks Pint and Rector, then deploys with Deployer on PHP 8.5 | `DEPLOY_SSH_PRIVATE_KEY` secret |

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

  larastan:
    uses: jcergolj/reusable-github-actions/.github/workflows/larastan.yml@master

  formatter:
    uses: jcergolj/reusable-github-actions/.github/workflows/code-formatter.yml@master
    secrets: inherit
```

## Deployer setup

The deploy workflow first runs `vendor/bin/pint --test` and `vendor/bin/rector process --dry-run --config=rector.php`. If either check fails, deployment stops. It then runs `vendor/bin/dep deploy production` from the GitHub Actions runner.

### 1. Prepare the application

Install and configure Deployer in the Laravel application:

```bash
composer require --dev jcergolj/metator-for-laravel
php artisan metator:install
```

Review `deploy.php`, set the production hostname, repository, deploy user, and deploy path. Bootstrap the server using the generated `scripts/server-bootstrap.sh` before using GitHub Actions.

The application must also have Laravel Pint and Rector installed, with a `rector.php` configuration file:

```bash
composer require --dev laravel/pint rector/rector
```

### 2. Add the deployment key

Create or use an SSH key whose public key is authorized for the production `deployer` user. Add the private key to the application repository as:

`Settings -> Secrets and variables -> Actions -> New repository secret`

Use the name `DEPLOY_SSH_PRIVATE_KEY`. Do not commit the private key.

### 3. Add the production environment

Create a `production` environment under `Settings -> Environments`. Add required reviewers if deployments need manual approval.

### 4. Call the deploy workflow

```yaml
jobs:
  tests:
    uses: jcergolj/reusable-github-actions/.github/workflows/tests.yml@master

  larastan:
    uses: jcergolj/reusable-github-actions/.github/workflows/larastan.yml@master

  deploy:
    needs: [tests, larastan]
    if: github.ref == 'refs/heads/master'
    uses: jcergolj/reusable-github-actions/.github/workflows/deploy.yml@master
    secrets:
      DEPLOY_SSH_PRIVATE_KEY: ${{ secrets.DEPLOY_SSH_PRIVATE_KEY }}
```

The deploy job uses PHP 8.5, installs Composer dependencies, verifies Pint and Rector without modifying files, loads the SSH key, and runs Deployer. The application must contain a working `deploy.php` and the server must be able to clone the repository using its configured GitHub deploy key.

## Security notes

- Pin workflow references to a release tag or commit SHA for production use instead of `@master`.
- Store `PAT_TOKEN` and `DEPLOY_SSH_PRIVATE_KEY` as GitHub secrets.
- Use a protected `production` environment for production deployments.
