# Plugin System
> Plugins, extensions, middleware, and declaration merging.

## Import
```ts
import { definePlugin, defineExtension, command, flag, arg } from "@seedcli/core"
import type { Middleware } from "@seedcli/core"
```

## Plugins

### `definePlugin(config)`
Package commands, extensions, templates, and config into a reusable npm package.

```ts
export default definePlugin({
  name: "docker",
  description: "Docker integration for Seed CLI",
  version: "1.0.0",
  seedcli: ">=1.0.0",
  commands: [
    command({
      name: "docker:build",
      flags: { tag: flag({ type: "string", alias: "t" }) },
      run: async (seed) => {
        await seed.system.exec(`docker build -t ${seed.flags.tag ?? "latest"} .`)
      },
    }),
  ],
  extensions: [
    { name: "docker-check", setup: async (seed) => { await seed.system.whichOrThrow("docker") } },
  ],
  templates: "./templates",
  defaults: { registry: "docker.io" },
})
```

### Plugin Options
| Option | Type | Description |
|--------|------|-------------|
| `name` | `string` | Plugin identifier (required) |
| `description` | `string` | Human-readable description |
| `version` | `string` | Plugin version (required) |
| `seedcli` | `string` | Required Seed CLI semver range |
| `peerPlugins` | `Record<string, string>` | Required peer plugin versions |
| `commands` | `Command[]` | Commands to register |
| `extensions` | `ExtensionConfig[]` | Extensions to register |
| `templates` | `string` | Path to template directory |
| `defaults` | `Record<string, unknown>` | Default config values |

### Loading Plugins
```ts
build("my-cli")
  .plugin("seedcli-plugin-docker")           // By name
  .plugin(["seedcli-plugin-docker", "seedcli-plugin-aws"]) // Multiple
  .plugins("./plugins")                       // From directory
```

### Publishing
Name your package `seedcli-plugin-<name>`, export plugin as default, declare `@seedcli/core` as peer dependency, include `"seedcli-plugin"` keyword.

## Extensions

### `defineExtension(config)`
Add setup/teardown lifecycle hooks and shared state.

```ts
export default defineExtension({
  name: "auth",
  description: "Authentication management",
  dependencies: ["config"],  // Optional: run after these extensions
  setup: async (seed) => {
    const token = seed.system.env("AUTH_TOKEN")
    seed.auth = { token, isAuthenticated: !!token }
  },
  teardown: async (seed) => {
    seed.print.debug("Auth extension cleaned up")
  },
})
```

### Extension Options
| Option | Type | Description |
|--------|------|-------------|
| `name` | `string` | Unique identifier (required) |
| `description` | `string` | Human-readable description |
| `dependencies` | `string[]` | Extensions that must run first |
| `setup` | `(seed: ExtensionSeed) => void \| Promise<void>` | Before command (required) |
| `teardown` | `(seed: ExtensionSeed) => void \| Promise<void>` | After command |

### Registration
Via builder (`.extension(ext)`), or auto-discovery by placing files in `extensions/` and using `.src()`.

### Dependencies
Resolved in topological order. Circular dependencies throw `ExtensionCycleError`.

### Declaration Merging
Type-safe extension state via TypeScript declaration merging:
```ts
declare module "@seedcli/core" {
  interface SeedExtensions {
    auth: { token: string | undefined; isAuthenticated: boolean }
    api: import("@seedcli/http").HttpClient
  }
}
```

Then `seed.auth.token` and `seed.api.get()` are fully typed in commands.

## Middleware

### Defining Middleware
```ts
const timing: Middleware = async (seed, next) => {
  const start = Date.now()
  await next()
  seed.print.muted(`Completed in ${Date.now() - start}ms`)
}
```

### Global vs Command-Level
```ts
// Global — runs for every command
build("my-cli").middleware(timing)

// Command-level — scoped to one command
command({ name: "deploy", middleware: [requireAuth], run: async (seed) => {} })
```

### Execution Order (Onion Model)
```
Global MW 1 → Global MW 2 → Command MW 1 → Command MW 2 → command.run(seed)
```
Each middleware wraps the next. Not calling `next()` short-circuits execution.

### Common Middleware Patterns
- **Error handling**: try/catch around `next()`, log or rethrow
- **Auth guard**: Check `seed.auth`, return early if unauthenticated
- **Timing**: Record start time, log elapsed after `next()`
- **Confirmation**: Prompt user before destructive actions
- **Analytics**: Track command usage with timing and status

## Lifecycle Order
1. Extension `setup` (dependency order: auth → api-client → metrics)
2. Global middleware
3. Command middleware
4. `command.run(seed)`
5. Extension `teardown` (reverse order: metrics → api-client → auth)
