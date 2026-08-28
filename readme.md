# Reusable GitHub Actions

Reusable GitHub Actions workflows for Laravel applications.

## Deployer deployment

The `deploy` workflow runs `vendor/bin/dep deploy production` on the GitHub Actions runner. It should be called only after the application test and static-analysis workflows pass.

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

Configure `DEPLOY_SSH_PRIVATE_KEY` with the private key whose public key is authorized for the production `deployer` user. The application must contain a configured `deploy.php`, and the server must already be bootstrapped for Deployer.

For production protection, configure required reviewers on the repository's `production` environment in GitHub.