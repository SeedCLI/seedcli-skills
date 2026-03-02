# @seedcli/core
> Builder API, commands, arguments, flags, routing, and runtime lifecycle.

## Import
```ts
import { build, command, arg, flag, run, defineConfig, definePlugin, defineExtension } from "@seedcli/core"
// or via umbrella
import { build, command, arg, flag, run } from "@seedcli/seed"
```

## Builder API

### `build(brand)`
Creates a new Builder instance. The `brand` string is your CLI's name used in help output and error messages.

```ts
const builder = build("my-cli")
```

### `.src(dir)`
Auto-discover commands and extensions from a directory.
```ts
builder.src(import.meta.dir)
// Discovers: src/commands/*.ts → commands, src/extensions/*.ts → extensions
```

### `.command(cmd)` / `.commands(cmds)`
Register commands manually.

### `.defaultCommand(cmd)`
Set a fallback command when no command is specified.

### `.plugin(name)` / `.plugins(dir)`
Load plugins by name (from node_modules) or discover from directory.
```ts
builder.plugin("seedcli-plugin-docker")
builder.plugins("./plugins")
```

### `.extension(ext)`
Register an extension.

### `.help(options?)` / `.noHelp()`
Enable/disable built-in help system. Options: `showAliases`, `showHidden`.

### `.version(version?)` / `.noVersion()`
Enable/disable --version flag. Auto-detects from package.json if no arg.

### `.debug()`
Enable --debug and --verbose flags. Sets `seed.meta.debug` to true.

### `.completions()`
Add shell completion subcommand (bash/zsh/fish/pwsh).

### `.middleware(fn)`
Add global middleware that runs before every command.

### `.config(options)`
Configure auto-loading of config files.

### `.onReady(fn)`
Hook that runs after setup but before command execution.

### `.onError(fn)`
Global error handler: `(error, seed) => void`.

### `.exclude(modules)`
Exclude modules for lighter CLIs: `builder.exclude(["http", "template"])`.

### `.create()`
Build and return the `Runtime` instance.

## Runtime

### `runtime.run(argv?)`
Execute the CLI. Uses process.argv by default, or pass custom argv for testing.

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
| `required` | `boolean` | Whether required (default: false) |
| `description` | `string` | Help text |
| `default` | `string \| number` | Default value |
| `choices` | `readonly string[]` | Restrict values (use `as const` for type narrowing) |
| `validate` | `(value) => boolean \| string` | Custom validation |

### `flag(options)`
Define a named flag.

| Option | Type | Description |
|--------|------|-------------|
| `type` | `"boolean" \| "string" \| "number" \| "string[]" \| "number[]"` | Flag type |
| `alias` | `string` | Short alias (e.g., "f" for --force) |
| `required` | `boolean` | Whether required |
| `default` | `unknown` | Default value |
| `description` | `string` | Help text |
| `choices` | `readonly string[]` | Restrict values |
| `validate` | `(value) => boolean \| string` | Custom validation |
| `hidden` | `boolean` | Hide from help output |

## Auto-Discovery

Place command files in `commands/` with `.src()`:
```
src/
├── commands/
│   ├── deploy.ts      → my-cli deploy
│   ├── init.ts        → my-cli init
│   └── db/
│       ├── migrate.ts → my-cli db migrate
│       └── seed.ts    → my-cli db seed
└── cli.ts
```
Each file should `export default` a `command()` call. Subdirectories become nested subcommands.

## Seed Context

Every command's `run` receives a typed `seed` object:
```ts
interface Seed<TArgs, TFlags> {
  args: TArgs
  flags: TFlags
  parameters: { raw: string[]; argv: string[]; command: string | undefined }
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
  meta: { version: string; commandName: string; brand: string; debug: boolean }
}
```

Modules are loaded lazily — only initialized when first accessed.

## Runtime Lifecycle

1. Register SIGINT/SIGTERM handlers
2. Strip --debug/--verbose flags (if enabled)
3. Parse raw argv
4. Load config files (if configured)
5. Discover and load plugins
6. Register extensions
7. Assemble the seed context (modules loaded lazily)
8. Run onReady hooks
9. Route to the matching command
10. Run extension setup functions (topological order)
11. Run global middleware
12. Run command-level middleware
13. Execute command.run(seed)
14. Run extension teardown (reverse order)
15. Cleanup and exit

## Types
```ts
type ArgDef = { type: "string" | "number"; required?: boolean; default?: unknown; choices?: readonly string[]; validate?: (v: unknown) => boolean | string; description?: string }
type FlagDef = { type: "boolean" | "string" | "number" | "string[]" | "number[]"; alias?: string; required?: boolean; default?: unknown; choices?: readonly string[]; validate?: (v: unknown) => boolean | string; hidden?: boolean; description?: string }
type Command = { name: string; description?: string; alias?: string[]; hidden?: boolean; args?: Record<string, ArgDef>; flags?: Record<string, FlagDef>; subcommands?: Command[]; middleware?: Middleware[]; run: (seed: Seed) => void | Promise<void> }
type Middleware = (seed: Seed, next: () => Promise<void>) => void | Promise<void>
```
