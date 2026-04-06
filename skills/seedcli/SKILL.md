---
name: seedcli
description: Generates code and provides API reference for Seed CLI, a batteries-included TypeScript-first CLI framework for Node.js. Covers all @seedcli/* packages including core builder, commands, plugins, extensions, prompts, filesystem, HTTP, templates, and testing. Use when working with seedcli, @seedcli/core, @seedcli/seed, building CLI tools for Node.js, or generating seedcli commands, plugins, and extensions.
---

# Seed CLI

Seed CLI is a batteries-included, modular, TypeScript-first CLI framework for **Node.js 24+**. It provides 18 packages under `@seedcli/*` covering command parsing, prompts, filesystem, HTTP, testing, build tooling, and distribution.

## Two Entry Patterns

**Builder pattern** (recommended for apps):
```ts
import { build } from "@seedcli/core"

const runtime = build("my-cli")
  .src(import.meta.dirname) // Auto-discovers commands/ and extensions/
  .version("1.0.0")
  .help()
  .debug()
  .create()

await runtime.run()
```

**Quick run** (scripts and simple CLIs):
```ts
import { run, command, arg } from "@seedcli/core"

await run({
  name: "my-cli",
  version: "1.0.0",
  commands: [
    command({
      name: "hello",
      args: { name: arg({ type: "string", required: true }) },
      run: ({ args, print }) => {
        print.info(`Hello, ${args.name}!`)
      },
    }),
  ],
})
```

## Project Setup

Create a new project:
```bash
npm create seedcli my-cli
# or: pnpm create seedcli my-cli
# or: yarn create seedcli my-cli
# or: bun create seedcli my-cli
# or: seed new my-cli
```

Standard Full template structure:
```text
my-cli/
├── src/
│   ├── commands/      # Auto-discovered by .src()
│   │   └── hello.ts
│   ├── extensions/    # Auto-discovered by .src()
│   │   └── timer.ts
│   └── index.ts       # Entry point with build().create()
├── tests/
│   └── hello.test.ts
├── .gitignore
├── biome.json
├── package.json
├── seed.config.ts
└── tsconfig.json
```

## Core Patterns

### Commands with Args and Flags

```ts
import { command, arg, flag } from "@seedcli/core"

export default command({
  name: "deploy",
  description: "Deploy the application",
  alias: ["d"],
  args: {
    environment: arg({
      type: "string",
      required: true,
      choices: ["staging", "production"] as const,
    }),
  },
  flags: {
    force: flag({ type: "boolean", alias: "f", description: "Skip confirmation" }),
    tag: flag({ type: "string", description: "Deploy tag" }),
    region: flag({
      type: "string",
      choices: ["us-east-1", "eu-west-1"] as const,
    }),
  },
  run: async ({ args, print }) => {
    print.info(`Deploying to ${args.environment}...`)
  },
})
```

**Arg types**: `"string"`, `"number"` with optional `required`, `default`, `choices`, `validate`.
**Flag types**: `"boolean"`, `"string"`, `"number"`, `"string[]"`, `"number[]"` with optional `alias`, `required`, `default`, `choices`, `validate`, `hidden`.

### Subcommands

Nested via `subcommands` or directory structure:
```ts
export default command({
  name: "db",
  subcommands: [migrateCommand, seedCommand],
})
// Usage: my-cli db migrate --up
```

Or via auto-discovery (`src/commands/db/migrate.ts` -> `my-cli db migrate`).

### The Seed Context

Every command's `run` receives a `seed` object with typed args/flags and lazily loaded modules:
```ts
interface Seed<TArgs, TFlags> {
  args: TArgs
  flags: TFlags
  parameters: { raw: string[]; argv: string[]; command: string | undefined }
  print: PrintModule
  prompt: PromptModule
  filesystem: typeof import("@seedcli/filesystem")
  system: typeof import("@seedcli/system")
  http: HttpModule
  template: TemplateModule
  patching: PatchingModule
  strings: StringsModule
  semver: typeof import("@seedcli/semver")
  packageManager: PackageManagerModule
  config: ConfigModule
  ui: typeof import("@seedcli/ui")
  completions: typeof import("@seedcli/completions")
  meta: { version: string; commandName: string; brand: string; debug: boolean }
}
```

### Builder API Chain

```ts
build("my-cli")
  .src(import.meta.dirname)        // Auto-discovers commands/ and extensions/
  .command(cmd)                    // Register one command manually
  .commands([cmd1, cmd2])          // Register multiple commands
  .defaultCommand(cmd)             // Fallback when no command is specified
  .extension(ext)                  // Register an extension
  .plugin(importedPlugin)          // Preferred for plugin type augmentation
  .plugin("seedcli-plugin-docker") // Or load by package name
  .plugins("./plugins", { matching: "my-*" })
  .help()                          // Enable --help
  .version()                       // Enable --version; auto-detect when .src() is set
  .debug()                         // Enable --debug / --verbose
  .completions()                   // Add shell completions subcommand
  .middleware(fn)                  // Global middleware
  .exclude(["http", "template"])   // Exclude unused modules
  .onReady(fn)                     // Hook before command execution
  .onError(fn)                     // Global error handler
  .create()
```

### Runtime Lifecycle

1. Register signal handlers
2. Strip `--debug` / `--verbose` flags when enabled
3. Handle `--version` / `--help` early exits
4. Discover and load plugins, register extensions
5. Route to the matching command
6. Parse command-specific args and flags
7. Assemble the seed context
8. Run `onReady` hooks
9. Run extension `setup` functions
10. Run global middleware -> command middleware -> `command.run(seed)`
11. Run extension `teardown` in reverse order
12. Cleanup

## Code Generation Templates

### CLI Entry Point
```ts
import { build } from "@seedcli/core"

const cli = build("{{name}}")
  .src(import.meta.dirname)
  .help()
  .version("0.1.0")
  .create()

await cli.run()
```

### Command
```ts
import { command } from "@seedcli/core"

export default command({
  name: "{{name}}",
  description: "TODO: describe the {{name}} command",
  run: async ({ print }) => {
    print.info("Running {{name}} command")
  },
})
```

### Extension
```ts
import { defineExtension } from "@seedcli/core"

export default defineExtension({
  name: "{{name}}",
  description: "TODO: describe the {{name}} extension",
  setup: async (seed) => {
    // Extend the seed context with your custom functionality
  },
})
```

Type-safe extension state via declaration merging:
```ts
declare module "@seedcli/core" {
  interface SeedExtensions {
    {{name}}: {
      // Extension state shape
    }
  }
}
```

### Plugin
```ts
import { definePlugin, command } from "@seedcli/core"

export default definePlugin({
  name: "{{name}}",
  description: "{{description}}",
  version: "1.0.0",
  seedcli: ">=1.0.0",
  commands: [
    command({
      name: "hello",
      run: async ({ print }) => {
        print.info("Hello from plugin")
      },
    }),
  ],
  templates: "./templates",
  defaults: { /* default config */ },
})
```

### Middleware
```ts
import type { Middleware } from "@seedcli/core"

const {{name}}: Middleware = async (seed, next) => {
  await next()
}
```

### Test
```ts
import { build } from "@seedcli/core"
import { createTestCli } from "@seedcli/testing"
import { describe, expect, test } from "vitest"

const runtime = build("test-cli").src("./src").create()

describe("{{command}} command", () => {
  test("executes successfully", async () => {
    const result = await createTestCli(runtime).run("{{command}} args")
    expect(result.exitCode).toBe(0)
  })

  test("supports env overrides", async () => {
    const result = await createTestCli(runtime)
      .env({ NODE_ENV: "test" })
      .run("{{command}}")
    expect(result.exitCode).toBe(0)
  })
})
```

`mockPrompt()`, `mockConfig()`, and `mockSystem()` are exposed on the test builder, but `env()` is the only helper that currently affects `run()`.

### Config File
```ts
import { defineConfig } from "@seedcli/core"

export default defineConfig({
  dev: {
    entry: "src/index.ts",
  },
  build: {
    entry: "src/index.ts",
    bundle: {
      outdir: "dist",
    },
    compile: {
      targets: ["node24-macos-arm64"],
    },
  },
})
```

## Package Reference

All packages can be imported directly or via the umbrella `@seedcli/seed` package.

| Package | Import | Reference |
|---------|--------|-----------|
| `@seedcli/core` | `build, command, arg, flag, run, definePlugin, defineExtension, defineConfig` | [references/core.md](references/core.md) |
| `@seedcli/print` | `info, success, warning, error, spin, table, box, tree, progressBar, colors` | [references/print.md](references/print.md) |
| `@seedcli/prompt` | `input, select, multiselect, confirm, password, number, editor, form, autocomplete` | [references/prompt.md](references/prompt.md) |
| `@seedcli/filesystem` | `read, write, readJson, writeJson, copy, move, find, exists, path, tmpDir` | [references/filesystem.md](references/filesystem.md) |
| `@seedcli/system` | `exec, shell, which, whichOrThrow, os, arch, env, open, isInteractive` | [references/system.md](references/system.md) |
| `@seedcli/http` | `get, post, put, patch, delete, create, createOpenAPIClient, download` | [references/http.md](references/http.md) |
| `@seedcli/template` | `generate, render, renderString, renderFile, directory` | [references/template.md](references/template.md) |
| `@seedcli/config` | `load, loadFile, get` | [references/config.md](references/config.md) |
| `@seedcli/strings` | `camelCase, pascalCase, snakeCase, kebabCase, plural, singular, truncate, template` | [references/strings.md](references/strings.md) |
| `@seedcli/patching` | `patch, append, prepend, exists, patchJson` | [references/patching.md](references/patching.md) |
| `@seedcli/semver` | `valid, satisfies, bump, compare, gt, lt, sort, maxSatisfying` | [references/semver.md](references/semver.md) |
| `@seedcli/package-manager` | `detect, detectFromUserAgent, getCommands, install, installDev, pmRunPrefix, remove, run, create` | [references/package-manager.md](references/package-manager.md) |
| `@seedcli/completions` | `bash, zsh, fish, powershell, install, detect` | [references/completions.md](references/completions.md) |
| `@seedcli/testing` | `createTestCli, mockSeed, createInterceptor` | [references/testing.md](references/testing.md) |
| `@seedcli/ui` | `header, status, list, countdown, divider, keyValue, progress, tree` | [references/ui.md](references/ui.md) |
| `@seedcli/seed` | All of the above (re-exported) | [references/seed-umbrella.md](references/seed-umbrella.md) |

Additional references:
- [Plugin System](references/plugin-system.md) — `definePlugin`, `defineExtension`, middleware, declaration merging
- [Build & Distribution](references/build-distribution.md) — `seed dev`, `seed build`, npm publishing, standalone binaries
- [Type Safety](references/type-safety.md) — `InferArgs`, `InferFlags`, `SeedExtensions` merging

## Common Recipes

See [examples/common-patterns.md](examples/common-patterns.md) for full recipes:
- **Interactive init** — Prompt-driven project initialization with file scaffolding
- **HTTP API wrapper** — Typed API client command with retry and error handling
- **File scaffolding** — Generate files from templates with patching for barrel exports
- **Version bumping** — Semver-aware version management with changelog updates

See [examples/project-scaffolds.md](examples/project-scaffolds.md) for scaffold-aligned project templates:
- **Minimal project** — Single-file CLI with inline command
- **Full project** — Auto-discovered commands, extensions, config, and tests
- **Plugin project** — Reusable plugin package with typed extensions and tests
