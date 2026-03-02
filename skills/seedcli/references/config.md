# @seedcli/config
> Load configuration files with automatic format detection.

## Import
```ts
import { load, loadFile, get } from "@seedcli/config"
import { defineConfig } from "@seedcli/core"
// Via seed context: seed.config.*
```

## Loading Config

```ts
const result = await seed.config.load<MyConfig>("myapp")
```

Searches in order: `myapp.config.ts`, `myapp.config.js`, `myapp.config.mjs`, `.myapprc`, `.myapprc.json`, `.myapprc.yaml`, `.myapprc.toml`, `package.json` → `myapp` field.

### Load Options

| Option | Type | Description |
|--------|------|-------------|
| `name` | `string` | Config name (required) |
| `defaults` | `Partial<T>` | Default values |
| `overrides` | `Partial<T>` | Override values (highest precedence) |
| `cwd` | `string` | Working directory |
| `envName` | `string` | Environment name for env-specific overrides |
| `dotenv` | `boolean` | Load .env files |
| `packageJson` | `boolean \| string` | Read from package.json field |
| `rcFile` | `boolean \| string` | Read RC file |
| `globalRc` | `boolean` | Check global RC file |

### Result

```ts
interface ResolvedConfig<T> {
  config: T             // Merged config object
  cwd: string           // Resolved working directory
  configFile?: string   // Path to config file found
  layers: ConfigLayer[] // All config layers
}
```

## Default Values

```ts
const { config } = await seed.config.load<AppConfig>("myapp", {
  defaults: { outputDir: "dist", verbose: false, plugins: [] },
})
```

## Loading a Specific File

```ts
const config = await seed.config.loadFile<{ port: number }>("config/server.json")
```

## Getting Nested Values

```ts
const port = seed.config.get(config, "server.port")
const host = seed.config.get(config, "server.host", "localhost")  // With default
```

## Builder Integration

```ts
build("my-cli")
  .config({ name: "myapp", defaults: { outputDir: "dist" } })
  .create()
```

User creates: `myapp.config.ts`
```ts
import { defineConfig } from "@seedcli/core"
export default defineConfig({ outputDir: "build", verbose: true })
```

## Environment-Specific Config

```ts
const { config } = await seed.config.load("myapp", { envName: "production" })
```

Searches for `myapp.config.production.ts` and merges over base config.

## Config Layers (precedence)

1. defaults (lowest) → 2. config file → 3. environment-specific → 4. package.json field (highest)
