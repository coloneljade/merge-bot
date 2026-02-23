# Merge Bot

Reusable GitHub Actions workflow that handles PR merging with automatic version bumping and CHANGELOG generation. Triggered by `/merge` comments on pull requests.

## What It Does

1. Reacts with :rocket: to acknowledge the command
2. Verifies commenter has write/admin access
3. Checks PR is open, not draft, and has no conflicts
4. Verifies all CI checks pass
5. Determines version bump from branch prefix (or override flag)
6. If bumping: reads version, computes new version, generates CHANGELOG entry, updates version file
7. Commits changes, waits for CI on new commit
8. Squash merges (or merge commit with `--no-squash`), deletes branch
9. Posts summary, reacts with :+1:

## Quick Start

Add this workflow to your repo:

```yaml
# .github/workflows/merge.yml
name: Merge Bot
on:
  issue_comment:
    types: [created]
concurrency:
  group: merge-${{ github.event.issue.number }}
  cancel-in-progress: false
jobs:
  merge:
    if: github.event.issue.pull_request && startsWith(github.event.comment.body, '/merge')
    uses: coloneljade/merge-bot/.github/workflows/merge.yml@v1
    with:
      bump-map: '{"feature":"minor","feat":"minor","fix":"patch"}'
    secrets:
      app-id: ${{ secrets.MERGE_APP_ID }}
      private-key: ${{ secrets.MERGE_APP_PRIVATE_KEY }}
```

## Commands

| Command | Behavior |
|---------|----------|
| `/merge` | Merge with auto-detected bump from branch prefix |
| `/merge --minor` | Force minor version bump |
| `/merge --patch` | Force patch version bump |
| `/merge --no-bump` | Merge without version bump or CHANGELOG |
| `/merge --no-squash` | Use merge commit instead of squash |

Flags can be combined: `/merge --patch --no-squash`

## Inputs Reference

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `bump-map` | string | `{"feature":"minor","feat":"minor","fix":"patch"}` | JSON mapping branch prefix to bump type |
| `version-file` | string | `Directory.Build.props` | Path to file containing version |
| `version-xpath` | string | `VersionPrefix` | XML element name holding the version |
| `changelog-file` | string | `CHANGELOG.md` | Path to CHANGELOG file |

## Secrets Reference

| Secret | Description |
|--------|-------------|
| `app-id` | GitHub App ID for `auldrant-merge-bot` |
| `private-key` | Private key (.pem) for the GitHub App |

Set these as repository secrets in each consuming repo under Settings > Secrets and variables > Actions.

## Branch Prefix Mapping

The `bump-map` input controls how branch prefixes map to version bumps. The prefix is everything before the first `/` in the branch name.

### Default Map

```json
{"feature": "minor", "feat": "minor", "fix": "patch"}
```

Branches not in the map (e.g., `docs/`, `chore/`, `ci/`, `test/`, `style/`) get no version bump and no CHANGELOG entry — they merge cleanly without touching version files.

### Example Maps

**Template package repo:**
```json
{"feature": "minor", "feat": "minor", "fix": "patch"}
```

**Application / library repo:**
```json
{"feature": "minor", "feat": "minor", "fix": "patch", "refactor": "minor", "perf": "patch"}
```

**Aggressive bumping (everything gets a version):**
```json
{"feature": "minor", "feat": "minor", "fix": "patch", "refactor": "patch", "perf": "patch", "docs": "patch"}
```

## CHANGELOG Generation

When a version file exists, the bot creates **versioned sections**:

```markdown
## [1.2.0] - 2026-02-23

### Added
- Description from PR title (#5)
```

When no version file exists, entries are appended to `## [Unreleased]` instead.

PR title format `type(scope): description` maps to sections:

| Type | Section |
|------|---------|
| `feat` | `### Added` |
| `fix` | `### Fixed` |
| `refactor`, `perf` | `### Changed` |
| `security` | `### Security` |

If the PR body contains a `## Summary` section with bullet points, those are used as entries instead of the title.

## GitHub Setup

### 1. Create the GitHub App

Go to **github.com > Settings > Developer settings > GitHub Apps > New GitHub App**.

| Field | Value |
|-------|-------|
| Name | `auldrant-merge-bot` |
| Homepage URL | `https://github.com/coloneljade/merge-bot` |
| Webhook | Uncheck "Active" |

**Repository permissions:**

| Permission | Access |
|------------|--------|
| Contents | Read & write |
| Pull requests | Read & write |
| Checks | Read-only |
| Issues | Read & write |

No user/org permissions needed. Subscribe to no events.

After creation:
1. Note the **App ID** from the app settings page
2. Under **Private keys**, generate a key and download the `.pem` file

### 2. Install the App

From the App settings page > **Install App** > select your account > choose repos (or "All repositories").

### 3. Add Repository Secrets

In each consuming repo: **Settings > Secrets and variables > Actions > New repository secret**

| Secret name | Value |
|-------------|-------|
| `MERGE_APP_ID` | The App ID number |
| `MERGE_APP_PRIVATE_KEY` | Full contents of the `.pem` file |

### 4. Create Branch Ruleset

In each consuming repo: **Settings > Rules > Rulesets > New branch ruleset**

| Setting | Value |
|---------|-------|
| Name | `main-protection` |
| Target branches | Include: `main` |
| Bypass list | Add: `auldrant-merge-bot` (App) |

**Rules to enable:**
- Restrict deletions
- Require a pull request before merging (1 approval)
- Require status checks to pass (add your CI workflow)
- Block force pushes

This ensures only the bot can push directly to `main`. Everyone else must go through PRs.

## Version Behavior

- **Major versions are never auto-bumped.** The bot only increments minor/patch within the current major version. Major bumps (e.g., `1.x.x` to `2.0.0`) are a manual process.
- The bot reads the current version from the configured `version-file` and `version-xpath`, computes the bump, writes it back, and commits.
- If no bump is needed (branch prefix not in map, or `--no-bump` flag), the bot skips CHANGELOG and version changes entirely.
- If no version file exists, the bot still generates CHANGELOG entries under `[Unreleased]` but skips version bumping.
- If no CHANGELOG file exists, the bot skips CHANGELOG generation entirely.
