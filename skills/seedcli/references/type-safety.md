# Type Safety
> TypeScript type inference patterns for args, flags, and extensions.

## Import
```ts
import type { InferArgs, InferFlags, Seed, SeedExtensions, Middleware } from "@seedcli/core"
```

## Arg Type Inference

### Basic Types
```ts
arg({ type: "string" })  // string | undefined
arg({ type: "number" })  // number | undefined
arg({ type: "string", required: true })  // string
arg({ type: "number", required: true })  // number
arg({ type: "string", default: "hello" })  // string (always defined)
```

### Choices Narrowing
Use `as const` to narrow to a union type:
```ts
arg({ type: "string", choices: ["dev", "staging", "prod"] as const })
// → "dev" | "staging" | "prod" | undefined

arg({ type: "string", required: true, choices: ["dev", "staging", "prod"] as const })
// → "dev" | "staging" | "prod"
```

## Flag Type Inference

```ts
flag({ type: "boolean" })        // boolean | undefined
flag({ type: "string" })         // string | undefined
flag({ type: "number" })         // number | undefined
flag({ type: "string[]" })       // string[] | undefined
flag({ type: "number[]" })       // number[] | undefined
flag({ type: "string", required: true })  // string
flag({ type: "boolean", default: false }) // boolean
flag({ type: "string", choices: ["json", "yaml"] as const }) // "json" | "yaml" | undefined
```

## InferArgs / InferFlags

Extract types from a command definition:
```ts
const myCommand = command({
  name: "deploy",
  args: {
    env: arg({ type: "string", required: true, choices: ["staging", "prod"] as const }),
  },
  flags: {
    force: flag({ type: "boolean" }),
    tag: flag({ type: "string" }),
  },
  run: async (seed) => {
    // seed.args.env → "staging" | "prod"
    // seed.flags.force → boolean | undefined
  },
})

type Args = InferArgs<typeof myCommand>   // { env: "staging" | "prod" }
type Flags = InferFlags<typeof myCommand> // { force: boolean | undefined; tag: string | undefined }
```

## SeedExtensions Declaration Merging

Extend the seed context with custom typed properties:
```ts
// src/types.ts
declare module "@seedcli/core" {
  interface SeedExtensions {
    auth: {
      token: string | undefined
      isAuthenticated: boolean
      user: { id: string; name: string } | null
    }
    db: {
      query: <T>(sql: string, params?: unknown[]) => Promise<T[]>
      close: () => Promise<void>
    }
  }
}
```

Now `seed.auth` and `seed.db` are fully typed everywhere:
```ts
command({
  name: "profile",
  run: async (seed) => {
    if (seed.auth.isAuthenticated) {
      const users = await seed.db.query<User>("SELECT * FROM users")
      seed.print.info(`Found ${users.length} users`)
    }
  },
})
```

## Typed Middleware
```ts
const requireAuth: Middleware = async (seed, next) => {
  if (!seed.auth?.isAuthenticated) {
    seed.print.error("Authentication required")
    return
  }
  await next()
}
```

## Common Patterns

### Typed command helper
```ts
function createCommand<A extends Record<string, ArgDef>, F extends Record<string, FlagDef>>(
  config: CommandConfig<A, F>
) {
  return command(config)
}
```

### Type-safe config
```ts
interface MyConfig {
  outputDir: string
  verbose: boolean
  plugins: string[]
}

const { config } = await seed.config.load<MyConfig>("myapp", {
  defaults: { outputDir: "dist", verbose: false, plugins: [] },
})
// config is fully typed as MyConfig
```
