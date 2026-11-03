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

## Hosted result (November 3, 2026)

The [initial install succeeds](https://github.com/karlhorky/repro-dependabot-pnpm12-lodash-workspace-security-updates-lockfile-only/actions/runs/41663352614).

Dependabot opens only [PR #1](https://github.com/karlhorky/repro-dependabot-pnpm12-lodash-workspace-security-updates-lockfile-only/pull/1), commit `bd08e7acf06eb7f79369baeba11f05996c49cae8`. It changes only `packages/app/package.json` from lodash 4.17.20 to 4.18.1. The [CI install fails](https://github.com/karlhorky/repro-dependabot-pnpm12-lodash-workspace-security-updates-lockfile-only/actions/runs/41664155755):

```text
$ pnpm install --frozen-lockfile --ignore-scripts
Scope: all 2 workspace projects
Error: ERR_PNPM_OUTDATED_LOCKFILE

  × installing dependencies
  ╰─▶ Cannot install with "frozen-lockfile" because pnpm-lock.yaml is not up
      to date with package.json.
      
        Failure reason:
        specifiers in the lockfile don't match specifiers in package.json:
      * in importers["packages/app"]:
      * 1 dependency is mismatched:
        - lodash (lockfile: 4.17.20, manifest: 4.18.1)
  help: Regenerate the lockfile with `pnpm install --lockfile-only` so that
        pnpm-lock.yaml reflects the current package.json, then re-run `pnpm
        install --frozen-lockfile`.

Process completed with exit code 1.
```

The separate [root security job fails](https://github.com/karlhorky/repro-dependabot-pnpm12-lodash-workspace-security-updates-lockfile-only/actions/runs/41663974361), creating no second PR. Its inputs include `command: security`, `security-updates-only: true`, `lockfile-only: true`, and `source.directories: ["/."]`. Relevant log lines:

```text
Requirements to unlock update_not_possible
Requirements update strategy lockfile_only
VulnerabilityAuditor: missing lockfile
The latest possible version of lodash that can be installed is 4.18.1
The earliest fixed version is 4.18.1.
```

It reports `security_update_not_possible` with `latest-resolvable-version: 4.18.1`, `lowest-non-vulnerable-version: 4.18.1`, and `conflicting-dependencies: []`.

The baseline reproduction without the strategy setting opens both a broken workspace PR and a root PR that updates both files and passes installation. Adding only `versioning-strategy: lockfile-only` reproduces a failed root security job and leaves only the broken workspace PR.

This result does not establish whether rejecting manifest changes under `lockfile-only` is intended for security updates. It demonstrates that the workspace job still creates an incomplete PR while the root job rejects the update.
