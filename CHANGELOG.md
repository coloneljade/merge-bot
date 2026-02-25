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