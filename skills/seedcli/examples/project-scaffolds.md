# Project Scaffolds
> Complete project templates for different use cases.

## Minimal Project

A single-file CLI with inline commands:

### `package.json`
```json
{
  "name": "my-cli",
  "version": "0.0.1",
  "type": "module",
  "bin": { "my-cli": "src/index.ts" },
  "dependencies": {
    "@seedcli/core": "latest"
  },
  "devDependencies": {
    "@types/bun": "latest"
  }
}
```

### `src/index.ts`
```ts
#!/usr/bin/env bun
import { run, command, arg, flag } from "@seedcli/core"

await run({
  name: "my-cli",
  version: "0.0.1",
  commands: [
    command({
      name: "greet",
      description: "Greet someone",
      args: { name: arg({ type: "string", required: true }) },
      flags: { shout: flag({ type: "boolean", alias: "s" }) },
      run: (seed) => {
        const msg = `Hello, ${seed.args.name}!`
        seed.print.info(seed.flags.shout ? msg.toUpperCase() : msg)
      },
    }),
  ],
})
```

## Full Project

A multi-command CLI with extensions, config, and tests:

### `package.json`
```json
{
  "name": "my-cli",
  "version": "0.0.1",
  "type": "module",
  "bin": { "my-cli": "src/index.ts" },
  "scripts": {
    "dev": "seed dev",
    "build": "seed build",
    "test": "bun test"
  },
  "dependencies": {
    "@seedcli/core": "latest",
    "@seedcli/seed": "latest"
  },
  "devDependencies": {
    "@seedcli/cli": "latest",
    "@seedcli/testing": "latest",
    "@types/bun": "latest"
  }
}
```

### `src/index.ts`
```ts
#!/usr/bin/env bun
import { build } from "@seedcli/core"

const runtime = build("my-cli")
  .src(import.meta.dir)
  .version("0.0.1")
  .help()
  .debug()
  .completions()
  .create()

await runtime.run()
```

### `src/commands/hello.ts`
```ts
import { command, arg, flag } from "@seedcli/core"

export default command({
  name: "hello",
  description: "Say hello",
  args: {
    name: arg({ type: "string", required: true, description: "Who to greet" }),
  },
  flags: {
    shout: flag({ type: "boolean", alias: "s", description: "SHOUT the greeting" }),
  },
  run: async (seed) => {
    const greeting = `Hello, ${seed.args.name}!`
    seed.print.info(seed.flags.shout ? greeting.toUpperCase() : greeting)
  },
})
```

### `src/extensions/timer.ts`
```ts
import { defineExtension } from "@seedcli/core"

export default defineExtension({
  name: "timer",
  description: "Track command execution time",
  setup: (seed) => {
    seed._startTime = Date.now()
  },
  teardown: (seed) => {
    const elapsed = Date.now() - seed._startTime
    seed.print.muted(`Done in ${elapsed}ms`)
  },
})
```

### `seed.config.ts`
```ts
import { defineConfig } from "@seedcli/core"

export default defineConfig({
  dev: { entry: "src/index.ts" },
  build: { entry: "src/index.ts", outDir: "dist", minify: true },
})
```

### `tests/hello.test.ts`
```ts
import { createTestCli } from "@seedcli/testing"
import { build } from "@seedcli/core"
import { describe, test, expect } from "bun:test"

const runtime = build("my-cli").src("./src").create()

describe("hello command", () => {
  test("greets by name", async () => {
    const result = await createTestCli(runtime).run("hello World")
    expect(result.stdout).toContain("Hello, World!")
    expect(result.exitCode).toBe(0)
  })

  test("shouts when flag is set", async () => {
    const result = await createTestCli(runtime).run("hello World --shout")
    expect(result.stdout).toContain("HELLO, WORLD!")
  })
})
```

## Plugin Project

A distributable plugin package:

### `package.json`
```json
{
  "name": "seedcli-plugin-example",
  "version": "1.0.0",
  "type": "module",
  "main": "src/index.ts",
  "keywords": ["seedcli-plugin"],
  "peerDependencies": {
    "@seedcli/core": ">=1.0.0"
  },
  "devDependencies": {
    "@seedcli/core": "latest",
    "@seedcli/testing": "latest",
    "@types/bun": "latest"
  }
}
```

### `src/index.ts`
```ts
import { definePlugin, command, arg, flag } from "@seedcli/core"

export default definePlugin({
  name: "example",
  description: "Example plugin for Seed CLI",
  version: "1.0.0",
  seedcli: ">=1.0.0",

  commands: [
    command({
      name: "example:hello",
      description: "Hello from the example plugin",
      args: { name: arg({ type: "string", default: "World" }) },
      run: async (seed) => {
        seed.print.info(`Hello from plugin: ${seed.args.name}!`)
      },
    }),
    command({
      name: "example:status",
      description: "Show plugin status",
      run: async (seed) => {
        seed.ui.header("Example Plugin", { subtitle: "v1.0.0", color: "cyan" })
        seed.ui.status("Plugin loaded", "success")
        seed.ui.status("Config valid", "success")
      },
    }),
  ],

  extensions: [
    {
      name: "example-init",
      description: "Initialize example plugin",
      setup: async (seed) => {
        seed.print.debug("Example plugin initialized")
        seed.example = { initialized: true }
      },
    },
  ],

  defaults: {
    greeting: "Hello",
  },
})
```

### `src/types.d.ts`
```ts
declare module "@seedcli/core" {
  interface SeedExtensions {
    example: {
      initialized: boolean
    }
  }
}
```

### `tests/plugin.test.ts`
```ts
import { createTestCli } from "@seedcli/testing"
import { build } from "@seedcli/core"
import { describe, test, expect } from "bun:test"

const runtime = build("test-cli")
  .plugin("./src")
  .create()

describe("example plugin", () => {
  test("hello command works", async () => {
    const result = await createTestCli(runtime).run("example:hello Alice")
    expect(result.stdout).toContain("Hello from plugin: Alice!")
    expect(result.exitCode).toBe(0)
  })
})
```
