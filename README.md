# Seed CLI Agent Skill

Agent skill for [Seed CLI](https://seedcli.dev) — a batteries-included, modular, TypeScript-first CLI framework for Node.js.

This skill helps AI coding agents understand Seed CLI APIs and generate seedcli-based code. It works across Claude Code, GitHub Copilot, Cursor, Cline, Gemini CLI, Codex, and any agent that supports the [skills.sh](https://skills.sh) ecosystem.

## Install

```bash
npx skills add SeedCLI/seedcli-skills
```

## What it does

- **API Reference** — Covers all 18 `@seedcli/*` packages with imports, types, and usage examples
- **Code Generation** — Templates for commands, plugins, extensions, middleware, tests, and config
- **Project Scaffolding** — Full project structures for minimal, full, and plugin projects
- **Progressive Disclosure** — General questions use the main skill file; specific package questions load targeted reference files

## Packages Covered

| Package | Description |
|---------|-------------|
| `@seedcli/core` | Builder API, commands, args, flags, routing, lifecycle |
| `@seedcli/print` | Logging, colors, spinner, table, box, tree, progress bar |
| `@seedcli/prompt` | input, select, multiselect, confirm, password, form |
| `@seedcli/filesystem` | Read/write, copy/move, find, path helpers, temp files |
| `@seedcli/system` | exec, shell, which, OS info, environment |
| `@seedcli/http` | HTTP client, retry, interceptors, OpenAPI typed client |
| `@seedcli/template` | Eta engine, generate, render, directory scaffolding |
| `@seedcli/config` | Config loading (ts/js/json/yaml/toml via c12) |
| `@seedcli/strings` | Case conversion, pluralize, truncate, template |
| `@seedcli/patching` | File patching, append, prepend, JSON patching |
| `@seedcli/semver` | Version utilities (parse, compare, bump, satisfy) |
| `@seedcli/package-manager` | PM detection, install, remove, run |
| `@seedcli/completions` | Shell completion generation (bash/zsh/fish/pwsh) |
| `@seedcli/testing` | createTestCli, mocks, interceptors |
| `@seedcli/ui` | header, status, list, countdown |
| `@seedcli/seed` | Umbrella re-export of all packages |
| `@seedcli/cli` | Build, dev, generate commands |

## License

MIT
