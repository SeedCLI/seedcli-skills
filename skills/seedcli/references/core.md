# @seedcli/core
> Builder API, commands, arguments, flags, routing, and runtime lifecycle.

## Import
```ts
import {
  build,
  command,
  arg,
  flag,
  run,
  defineConfig,
  definePlugin,
  defineExtension,
} from "@seedcli/core"
// or via umbrella
import { build, command, arg, flag, run } from "@seedcli/seed"
```

## Builder API

### `build(brand)`
Creates a new Builder instance. The `brand` string is your CLI name and is used in help output, error messages, and metadata.

```ts
const builder = build("my-cli")
```

### `.src(dir)`
Auto-discover commands and extensions from a directory.

```ts
builder.src(import.meta.dirname)
// Discovers:
//   src/commands/*.ts   -> commands
//   src/extensions/*.ts -> extensions
```

### `.command(cmd)` / `.commands(cmds)`
Register commands manually.

### `.defaultCommand(cmd)`
Set a fallback command when no command is specified.

### `.plugin(source)` / `.plugins(dir, options?)`
Load plugins by imported module, package name, or discover them from a directory.

```ts
builder.plugin(importedPlugin)
builder.plugin("seedcli-plugin-docker")
builder.plugins("./plugins", { matching: "my-*" })
```

### `.extension(ext)`
Register an extension.

### `.help(options?)` / `.noHelp()`
Enable or disable the built-in help system. Options: `showAliases`, `showHidden`.

### `.version(version?)` / `.noVersion()`
Enable or disable the `--version` flag. If you call `.version()` without a string, Seed walks up from the entry script's location on disk to find the nearest `package.json` and reads its `version` field. This works in dev mode (`bun src/index.ts`), in bundled output (`node dist/index.js`), and for globally-installed CLIs — no need to import `package.json` yourself. (Fixed in v1.1.5; earlier versions only auto-detected when `.src()` was set, which silently broke in bundled mode.)

### `.debug()`
Enable `--debug` and `--verbose`. When active, `seed.meta.debug` is `true`.

### `.completions()`
Add the `completions` subcommand for bash, zsh, fish, and PowerShell.

### `.middleware(fn)`
Add global middleware that runs before every command.

### `.onReady(fn)`
Hook that runs after the seed context is assembled, before extension setup and command execution.

### `.onError(fn)`
Global error handler: `(error, seed) => void`.

### `.exclude(modules)`
Exclude built-in modules for lighter CLIs: `builder.exclude(["http", "template"])`.

### `.create()`
Build and return the `Runtime` instance.

## Runtime

### `runtime.run(argv?)`
Execute the CLI. Uses `process.argv` by default, or pass custom argv for testing.

## The `run()` Shortcut
```ts
await run({
  name: "my-cli",
  version: "1.0.0",
  commands: [/* ... */],
})
```

## Commands

### `command(config)`
Define a command with typed args, flags, and a run handler.

Command options:
| Option | Type | Description |
|--------|------|-------------|
| `name` | `string` | Command name (required) |
| `description` | `string` | Help text |
| `alias` | `string[]` | Command aliases |
| `hidden` | `boolean` | Hide from help output |
| `args` | `Record<string, ArgDef>` | Positional arguments |
| `flags` | `Record<string, FlagDef>` | Named flags |
| `subcommands` | `Command[]` | Nested subcommands |
| `middleware` | `Middleware[]` | Command-specific middleware |
| `run` | `(seed: Seed) => void \| Promise<void>` | Command handler |

### `arg(options)`
Define a positional argument.

| Option | Type | Description |
|--------|------|-------------|
| `type` | `"string" \| "number"` | Argument type |
| `required` | `boolean` | Whether required |
| `description` | `string` | Help text |
| `default` | `string \| number` | Default value |
| `choices` | `readonly string[]` | Restrict values |
| `validate` | `(value) => boolean \| string` | Custom validation |

### `flag(options)`
Define a named flag.

| Option | Type | Description |
|--------|------|-------------|
| `type` | `"boolean" \| "string" \| "number" \| "string[]" \| "number[]"` | Flag type |
| `alias` | `string` | Short alias (for example `"f"` for `--force`) |
| `required` | `boolean` | Whether required |
| `default` | `unknown` | Default value |
| `description` | `string` | Help text |
| `choices` | `readonly string[]` | Restrict values |
| `validate` | `(value) => boolean \| string` | Custom validation |
| `hidden` | `boolean` | Hide from help output |

## Auto-Discovery

Place command files in `commands/` and extension files in `extensions/`, then point `.src()` at the CLI entry directory:

```text
src/
├── commands/
│   ├── deploy.ts      -> my-cli deploy
│   ├── init.ts        -> my-cli init
│   └── db/
│       ├── migrate.ts -> my-cli db migrate
│       └── seed.ts    -> my-cli db seed
├── extensions/
│   └── timer.ts
└── index.ts
```

Each command file should `export default command(...)`. Subdirectories become nested subcommands.

## Seed Context

Every command `run` receives a typed `seed` object:
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

Modules are loaded lazily and can be excluded with `.exclude(...)`.

## Runtime Lifecycle

1. Register SIGINT/SIGTERM handlers
2. Strip `--debug` / `--verbose` flags when enabled
3. Handle `--version` / `--help` early exits
4. Discover and load plugins, register extensions
5. Route to the matching command
6. Parse command-specific args and flags
7. Assemble the seed context
8. Run `onReady` hooks
9. Run extension `setup` functions
10. Run global middleware
11. Run command-level middleware
12. Execute `command.run(seed)`
13. Run extension `teardown` in reverse order
14. Cleanup and exit

## Types
```ts
type ArgDef = {
  type: "string" | "number"
  required?: boolean
  default?: unknown
  choices?: readonly string[]
  validate?: (v: unknown) => boolean | string
  description?: string
}

type FlagDef = {
  type: "boolean" | "string" | "number" | "string[]" | "number[]"
  alias?: string
  required?: boolean
  default?: unknown
  choices?: readonly string[]
  validate?: (v: unknown) => boolean | string
  hidden?: boolean
  description?: string
}

type Middleware = (seed: Seed, next: () => Promise<void>) => void | Promise<void>
```
