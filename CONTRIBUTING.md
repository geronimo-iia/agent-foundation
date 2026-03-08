# Contributing

## Adding or updating a document

All `.md` files (except `README.md`, `CHANGELOG.md`, `CONTRIBUTING.md`, `ARCHIVED.md`) must include YAML frontmatter. Follow the authoring standard:

- [Documentation Authoring](docs/authoring-guide.md) — required fields (`title`, `summary`, `read_when`), field rules, and examples
- [Specification Organization](docs/organization.md) — where to place new documents, naming conventions, domain placement workflow

## Adding a new domain

1. Create a new folder with a `definition.md` — define the domain scope first
2. Add domain-specific specifications following the authoring guide
3. Update the structure in `README.md`

## Commit format

Follow the semantic commit standard from [agent-software](../agent-software/version-control-release/git-commit-semantic.md).

## Index

`index.json` is auto-generated — do not edit it manually. It is regenerated on every push to `main`.
