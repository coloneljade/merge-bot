# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]
### Added

- Support JSON version files (e.g., `package.json`) with auto-detection by file extension
- Auto-update `@v1` tag to latest `main` on every push

### Fixed

- Add Readiness Check section to README with setup instructions ([#5](https://github.com/coloneljade/merge-bot/pull/5))
- Document the permissions requirement for caller workflows ([#5](https://github.com/coloneljade/merge-bot/pull/5))
- Preserve JSON formatting when bumping version ([#8](https://github.com/coloneljade/merge-bot/pull/8))
- Replace raw regex with `npm version` for `package.json` version updates ([#9](https://github.com/coloneljade/merge-bot/pull/9))
- XML files keep the existing regex approach unchanged ([#9](https://github.com/coloneljade/merge-bot/pull/9))
- Uses `--no-git-tag-version`, `--package-lock=false`, and `--prefix` flags ([#9](https://github.com/coloneljade/merge-bot/pull/9))
- Replace single point-in-time CI check with poll-and-wait loop (30s interval, 10min timeout) ([#12](https://github.com/coloneljade/merge-bot/pull/12))
- Log which checks are still running during each poll cycle ([#12](https://github.com/coloneljade/merge-bot/pull/12))
- Reuses the same pattern already used for bump commit CI waiting (Step 12) ([#12](https://github.com/coloneljade/merge-bot/pull/12))
- Remove CI check step from readiness workflow ([#14](https://github.com/coloneljade/merge-bot/pull/14))
- Remove CI status row from readiness PR comment ([#14](https://github.com/coloneljade/merge-bot/pull/14))
- GitHub's native checks UI already shows CI status inline ([#14](https://github.com/coloneljade/merge-bot/pull/14))
- Comment now focuses on bot-specific info: draft status, conflicts, version preview, /merge CTA ([#14](https://github.com/coloneljade/merge-bot/pull/14))