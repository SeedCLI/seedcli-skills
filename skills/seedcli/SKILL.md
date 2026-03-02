---
name: seedcli
description: Generates code and provides API reference for Seed CLI, a batteries-included TypeScript-first CLI framework powered by Bun. Covers all @seedcli/* packages including core builder, commands, plugins, extensions, prompts, filesystem, HTTP, templates, and testing. Use when working with seedcli, @seedcli/core, @seedcli/seed, building CLI tools with Bun, or generating seedcli commands, plugins, and extensions.
---

# Seed CLI

Seed CLI is a batteries-included, modular, TypeScript-first CLI framework powered by **Bun** (>=1.3.9). It provides 18 packages under `@seedcli/*` covering everything from command parsing to HTTP clients, file operations, interactive prompts, and testing.

## Two Entry Patterns

**Builder pattern** (recommended for apps):
```ts
import { build } from "@seedcli/core"

const runtime = build("my-cli")
  .src(import.meta.dir) // Auto-discover commands/ and extensions/
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
      run: (seed) => seed.print.info(`Hello, ${seed.args.name}!`),
    }),
  ],
})
```

## Project Setup

Create a new project:
```bash
bun create seedcli my-cli
# or: seed new my-cli
```

Standard project structure:
```
my-cli/
├── src/
│   ├── commands/      # Auto-discovered by .src()
│   │   └── hello.ts
│   ├── extensions/    # Auto-discovered by .src()
│   │   └── timer.ts
│   └── index.ts       # Entry point with build().create()
├── tests/
│   └── hello.test.ts
├── seed.config.ts     # Dev/build configuration
├── package.json
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
  run: async (seed) => {
    // seed.args.environment → "staging" | "production" (fully typed)
    // seed.flags.force → boolean | undefined
    // seed.flags.region → "us-east-1" | "eu-west-1" | undefined
    seed.print.info(`Deploying to ${seed.args.environment}...`)
  },
})
```

**Arg types**: `"string"`, `"number"` — with optional `required`, `default`, `choices`, `validate`.
**Flag types**: `"boolean"`, `"string"`, `"number"`, `"string[]"`, `"number[]"` — with optional `alias`, `required`, `default`, `choices`, `validate`, `hidden`.

### Subcommands

Nested via `subcommands` option or directory structure:
```ts
export default command({
  name: "db",
  subcommands: [migrateCommand, seedCommand],
})
// Usage: my-cli db migrate --up
```

Or via auto-discovery (`src/commands/db/migrate.ts` → `my-cli db migrate`).

### The Seed Context

Every command's `run` receives a `seed` object with typed args/flags and lazily-loaded modules:
```ts
interface Seed<TArgs, TFlags> {
  args: TArgs
  flags: TFlags
  parameters: { raw: string[]; argv: string[]; command: string | undefined }
  // Modules (lazy-loaded)
  print: PrintModule
  prompt: PromptModule
  filesystem: FilesystemModule
  system: SystemModule
  http: HttpModule
  template: TemplateModule
  patching: PatchingModule
  strings: StringsModule
  semver: SemverModule
  packageManager: PackageManagerModule
  config: ConfigModule
  // Metadata
  meta: { version: string; commandName: string; brand: string; debug: boolean }
}
```

### Builder API Chain

```ts
build("my-cli")
  .src(import.meta.dir)           // Auto-discover commands/ and extensions/
  .command(cmd)                    // Register command manually
  .commands([cmd1, cmd2])          // Register multiple commands
  .defaultCommand(cmd)             // Fallback when no command specified
  .extension(ext)                  // Register extension
  .plugin("seedcli-plugin-docker") // Load plugin by name
  .plugins("./plugins")            // Discover plugins from directory
  .help()                          // Enable --help flag
  .version("1.0.0")               // Enable --version flag
  .debug()                         // Enable --debug/--verbose flags
  .completions()                   // Enable shell completions subcommand
  .middleware(fn)                  // Global middleware
  .config({ name: "myapp" })      // Auto-load config files
  .exclude(["http", "template"])   // Exclude unused modules
  .onReady(fn)                     // Hook: after setup, before command
  .onError(fn)                     // Global error handler
  .create()                        // Build the Runtime
```

### Runtime Lifecycle

1. Register signal handlers → 2. Parse argv → 3. Load config → 4. Load plugins → 5. Register extensions → 6. Assemble seed context → 7. `onReady` hooks → 8. Route to command → 9. Extension `setup` (topological order) → 10. Global middleware → 11. Command middleware → 12. `command.run(seed)` → 13. Extension `teardown` (reverse order) → 14. Cleanup

## Code Generation Templates

### CLI Entry Point
```ts
import { build } from "@seedcli/core"

const runtime = build("{{name}}")
  .src(import.meta.dir)
  .version("0.0.1")
  .help()
  .debug()
  .create()

await runtime.run()
```

### Command
```ts
import { command, arg, flag } from "@seedcli/core"

export default command({
  name: "{{name}}",
  description: "{{description}}",
  args: {
    // name: arg({ type: "string", required: true }),
  },
  flags: {
    // verbose: flag({ type: "boolean", alias: "v" }),
  },
  run: async (seed) => {
    // Implementation
  },
})
```

### Extension
```ts
import { defineExtension } from "@seedcli/core"

export default defineExtension({
  name: "{{name}}",
  description: "{{description}}",
  // dependencies: ["other-ext"],  // Run after these
  setup: async (seed) => {
    // Initialize shared state
    seed.{{name}} = { /* ... */ }
  },
  teardown: async (seed) => {
    // Cleanup resources
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
import { definePlugin, command, flag } from "@seedcli/core"

export default definePlugin({
  name: "{{name}}",
  description: "{{description}}",
  version: "1.0.0",
  seedcli: ">=1.0.0",
  commands: [
    command({
      name: "{{name}}:run",
      run: async (seed) => { /* ... */ },
    }),
  ],
  extensions: [
    { name: "{{name}}-check", setup: async (seed) => { /* ... */ } },
  ],
  templates: "./templates",
  defaults: { /* default config */ },
})
```

### Middleware
```ts
import type { Middleware } from "@seedcli/core"

const {{name}}: Middleware = async (seed, next) => {
  // Before command
  await next() // Run next middleware or command
  // After command
}
```

### Test
```ts
import { createTestCli } from "@seedcli/testing"
import { describe, test, expect } from "bun:test"

const runtime = build("test-cli").src("./src").create()

describe("{{command}} command", () => {
  test("executes successfully", async () => {
    const result = await createTestCli(runtime).run("{{command}} args")
    expect(result.exitCode).toBe(0)
    expect(result.stdout).toContain("expected output")
  })

  test("handles interactive prompts", async () => {
    const result = await createTestCli(runtime)
      .mockPrompt({ "Question?": "answer" })
      .mockConfig({ key: "value" })
      .run("{{command}}")
    expect(result.exitCode).toBe(0)
  })
})
```

### Config File
```ts
import { defineConfig } from "@seedcli/core"

export default defineConfig({
  dev: {
    entry: "src/index.ts",
  },
  build: {
    entry: "src/index.ts",
    outDir: "dist",
    minify: true,
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
| `@seedcli/config` | `load, loadFile, get, defineConfig` | [references/config.md](references/config.md) |
| `@seedcli/strings` | `camelCase, pascalCase, snakeCase, kebabCase, plural, singular, truncate, template` | [references/strings.md](references/strings.md) |
| `@seedcli/patching` | `patch, append, prepend, exists, patchJson` | [references/patching.md](references/patching.md) |
| `@seedcli/semver` | `valid, satisfies, bump, compare, gt, lt, sort, maxSatisfying` | [references/semver.md](references/semver.md) |
| `@seedcli/package-manager` | `detect, install, installDev, remove, run, create` | [references/package-manager.md](references/package-manager.md) |
| `@seedcli/completions` | `bash, zsh, fish, powershell, install, detect` | [references/completions.md](references/completions.md) |
| `@seedcli/testing` | `createTestCli, mockSeed, createInterceptor` | [references/testing.md](references/testing.md) |
| `@seedcli/ui` | `header, status, list, countdown` | [references/ui.md](references/ui.md) |
| `@seedcli/seed` | All of the above (re-exported) | [references/seed-umbrella.md](references/seed-umbrella.md) |

Additional references:
- [Plugin System](references/plugin-system.md) — `definePlugin`, `defineExtension`, middleware, declaration merging
- [Build & Distribution](references/build-distribution.md) — Bundle, compile, multi-platform targets
- [Type Safety](references/type-safety.md) — `InferArgs`, `InferFlags`, `SeedExtensions` merging

## Common Recipes

See [examples/common-patterns.md](examples/common-patterns.md) for full recipes:
- **Interactive init** — Prompt-driven project initialization with file scaffolding
- **HTTP API wrapper** — Create a typed API client command with retry and error handling
- **File scaffolding** — Generate files from templates with patching for barrel exports
- **Version bumping** — Semver-aware version management with changelog updates

See [examples/project-scaffolds.md](examples/project-scaffolds.md) for complete project templates:
- **Minimal project** — Single-file CLI with inline commands
- **Full project** — Multi-command CLI with extensions, config, and tests
- **Plugin project** — Distributable plugin package with commands and templates
