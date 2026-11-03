# pnpm workspace security updates with lodash and lockfile-only

Compare with https://github.com/karlhorky/repro-dependabot-pnpm12-workspace-security-updates-missed-lockfile-update.

The only functional change is adding `versioning-strategy: lockfile-only` to `.github/dependabot.yml`. The manifests, lockfile, workspace configuration, and CI workflow are identical to that reproduction.

One workspace member pins `lodash@4.17.20`. The root pins pnpm 12.11.2. CI uses Node.js 24 and runs `pnpm install --frozen-lockfile --ignore-scripts`.

## Reproduce

1. Publish these files in a public GitHub repository
2. Enable Dependabot alerts and security updates
3. Inspect the alerts, security update jobs, generated PRs, and CI results

Scheduled version PRs are disabled to isolate security updates.

Expected: one security PR updates the workspace manifest and shared root lockfile together and passes CI.
