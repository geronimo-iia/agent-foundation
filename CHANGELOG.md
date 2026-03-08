# Changelog

All notable changes to this project will be documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [Unreleased]

## [0.2.1] - 2026-07-16

### Changed

- GitHub Pages navigation structure improved with automatic folder-based organization
- Added logical nav_order for sections: Bootstrap → Documentation → Skills → Agents → Tools → Information Flow → Rules → Schemas
- Enabled jekyll plugins for better automatic navigation

## [0.2.0] - 2026-07-15

### Added
- `CHANGELOG.md`, `CONTRIBUTING.md`, `LICENSE` (MIT)
- `rules/` — workspace rules best practices, templates, and AI system portability guide (moved from `agent-software`)
- `docs/authoring-guide.md` — clarified frontmatter exclusions (README, CHANGELOG, CONTRIBUTING, ARCHIVED)

### Changed
- `README.md` — added maturity table, full structure, key specifications, license and contributing links
- `bootstrap/development-environment.md` — updated to reference `agent-software`, fixed rule templates path
- `agentctl.toml` — updated hub id to `geronimo-iia/agent-foundation`
- CI workflow — added `if` guard on commit step, added `--hub-id` flag

## [0.1.0] - 2025-01-16

### Added
- Initial specifications: `skills/`, `docs/`, `agents/`, `tools/`, `information-flow/`, `bootstrap/`
- JSON schemas in `schemas/`
- `index.json` generated via `agentctl`
- CI workflow publishing `index.json` on push to `main`
- `agentctl.toml` with hub id `geronimo-iia/agent-foundation`
