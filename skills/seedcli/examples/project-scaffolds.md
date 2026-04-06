# Project Scaffolds
> Representative scaffold excerpts aligned with the current Seed project structure and tooling.

## Minimal Project

A single-file CLI with an inline command and a Vitest smoke test.

### `package.json`
```json
{
  "name": "my-cli",
  "version": "0.1.0",
  "type": "module",
  "bin": {
    "my-cli": "./src/index.ts"
  },
  "scripts": {
    "dev": "seed dev",
    "test": "vitest run",
    "build": "seed build",
    "compile": "seed build --compile --outfile my-cli",
    "lint": "biome check .",
    "lint:fix": "biome check --write .",
    "format": "biome format --write .",
    "typecheck": "tsc --noEmit"
  },
  "engines": {
    "node": ">=24.0.0"
  },
  "dependencies": {
    "@seedcli/core": "latest"
  },
  "devDependencies": {
    "@biomejs/biome": "^2.4.4",
    "@seedcli/cli": "latest",
    "@types/node": "^24.0.0",
    "tsx": "^4.19.0",
    "typescript": "^5.9.0",
    "vitest": "^3.2.0"
  }
}
```

### `src/index.ts`
```ts
#!/usr/bin/env node

import { build, command } from "@seedcli/core"

const hello = command({
  name: "hello",
  description: "Say hello",
  run: async ({ print }) => {
    print.info("Hello from my-cli!")
  },
})

const cli = build("my-cli")
  .command(hello)
  .help()
  .version("0.1.0")
  .create()

await cli.run()
```

### `tests/index.test.ts`
```ts
import { describe, expect, test } from "vitest"
import { execFile } from "node:child_process"
import { join } from "node:path"
import { promisify } from "node:util"

const execFileAsync = promisify(execFile)

describe("my-cli", () => {
  test("hello command prints greeting", async () => {
    const { stdout } = await execFileAsync("node", [
      "--import", "tsx",
      join(import.meta.dirname, "..", "src", "index.ts"),
      "hello",
    ])

    expect(stdout).toContain("Hello from my-cli!")
  })
})
```

## Full Project

An auto-discovered CLI with commands, extensions, `seed.config.ts`, and tests.

### `package.json`
```json
{
  "name": "my-cli",
  "version": "0.1.0",
  "type": "module",
  "bin": {
    "my-cli": "./src/index.ts"
  },
  "scripts": {
    "dev": "seed dev",
    "build": "seed build",
    "compile": "seed build --compile --outfile my-cli",
    "test": "vitest run",
    "lint": "biome check .",
    "lint:fix": "biome check --write .",
    "format": "biome format --write .",
    "typecheck": "tsc --noEmit"
  },
  "engines": {
    "node": ">=24.0.0"
  },
  "dependencies": {
    "@seedcli/core": "latest"
  },
  "devDependencies": {
    "@biomejs/biome": "^2.4.4",
    "@seedcli/cli": "latest",
    "@seedcli/testing": "latest",
    "@types/node": "^24.0.0",
    "tsx": "^4.19.0",
    "typescript": "^5.9.0",
    "vitest": "^3.2.0"
  }
}
```

### `src/index.ts`
```ts
#!/usr/bin/env node

import { build } from "@seedcli/core"

const cli = build("my-cli")
  .src(import.meta.dirname)
  .help()
  .version("0.1.0")
  .create()

await cli.run()
```

### `src/commands/hello.ts`
```ts
import { command, arg, flag } from "@seedcli/core"

export default command({
  name: "hello",
  description: "Say hello",
  args: {
    name: arg({ type: "string", description: "Who to greet" }),
  },
  flags: {
    loud: flag({ type: "boolean", default: false, alias: "l", description: "SHOUT the greeting" }),
    timed: flag({ type: "boolean", default: false, alias: "t", description: "Show execution time" }),
  },
  run: async ({ args, flags, print, timer }) => {
    if (flags.timed) timer.start()

    const name = args.name ?? "World"
    const greeting = `Hello, ${name}!`
    print.info(flags.loud ? greeting.toUpperCase() : greeting)

    if (flags.timed) print.muted(`Done in ${timer.stop()}`)
  },
})
```

### `src/extensions/timer.ts`
```ts
import { defineExtension } from "@seedcli/core"

declare module "@seedcli/core" {
  interface SeedExtensions {
    timer: {
      start(): void
      stop(): string
    }
  }
}

export default defineExtension({
  name: "timer",
  description: "Simple timing utility",
  setup: (seed) => {
    let startTime = 0

    seed.timer = {
      start: () => {
        startTime = performance.now()
      },
      stop: () => {
        const elapsed = performance.now() - startTime
        return `${elapsed.toFixed(0)}ms`
      },
    }
  },
})
```

### `seed.config.ts`
```ts
import { defineConfig } from "@seedcli/core"

export default defineConfig({
  dev: {
    entry: "src/index.ts",
    clearScreen: true,
  },
  // build: {
  //   entry: "src/index.ts",
  //   bundle: {
  //     outdir: "dist",
  //   },
  //   compile: {
  //     targets: ["node24-macos-arm64"],
  //   },
  // },
})
```

### `tests/hello.test.ts`
```ts
import { describe, expect, test } from "vitest"
import { execFile } from "node:child_process"
import { promisify } from "node:util"

const execFileAsync = promisify(execFile)

describe("my-cli CLI", () => {
  test("hello command runs", async () => {
    const { stdout } = await execFileAsync("node", [
      "--import", "tsx",
      "src/index.ts",
      "hello",
    ])

    expect(stdout).toContain("Hello, World!")
  })
})
```

## Plugin Project

A reusable plugin package for npm distribution.

### `package.json`
```json
{
  "name": "my-plugin",
  "version": "0.1.0",
  "type": "module",
  "main": "./src/index.ts",
  "types": "./src/index.ts",
  "exports": {
    ".": {
      "import": "./src/index.ts",
      "types": "./src/index.ts"
    }
  },
  "files": ["dist", "LICENSE"],
  "scripts": {
    "build": "tsc --project tsconfig.build.json",
    "prepublishOnly": "npm run build",
    "test": "vitest run",
    "lint": "biome check .",
    "lint:fix": "biome check --write .",
    "format": "biome format --write .",
    "typecheck": "tsc --noEmit"
  },
  "engines": {
    "node": ">=24.0.0"
  },
  "dependencies": {
    "@seedcli/core": "latest"
  },
  "peerDependencies": {
    "@seedcli/core": "latest"
  },
  "devDependencies": {
    "@biomejs/biome": "^2.4.4",
    "@seedcli/testing": "latest",
    "@types/node": "^24.0.0",
    "typescript": "^5.9.0",
    "vitest": "^3.2.0"
  },
  "publishConfig": {
    "main": "./dist/index.js",
    "types": "./dist/index.d.ts",
    "exports": {
      ".": {
        "import": "./dist/index.js",
        "types": "./dist/index.d.ts"
      }
    }
  }
}
```

### `src/index.ts`
```ts
import { definePlugin } from "@seedcli/core"
import helloCommand from "./commands/hello.js"
import { exampleExtension } from "./extensions/example.js"

export type {} from "./types.js"

export default definePlugin({
  name: "my-plugin",
  version: "0.1.0",
  description: "Example plugin",
  commands: [helloCommand],
  extensions: [exampleExtension],
})
```

### `src/commands/hello.ts`
```ts
import { command, arg, flag } from "@seedcli/core"

export default command({
  name: "hello",
  description: "Say hello",
  args: {
    name: arg({ type: "string", description: "Who to greet" }),
  },
  flags: {
    loud: flag({ type: "boolean", default: false, alias: "l", description: "SHOUT the greeting" }),
  },
  run: async ({ args, flags, print }) => {
    const name = args.name ?? "World"
    const greeting = `Hello, ${name}!`
    print.info(flags.loud ? greeting.toUpperCase() : greeting)
  },
})
```

### `src/extensions/example.ts`
```ts
import { defineExtension } from "@seedcli/core"

export const exampleExtension = defineExtension({
  name: "my-plugin",
  description: "Example extension for my-plugin",
  setup: (seed) => {
    seed.print.muted("[my-plugin] extension loaded")
    seed.my_plugin = {
      greet: (name: string) => `Hello from my-plugin, ${name}!`,
    }
  },
})
```

### `src/types.d.ts`
```ts
declare module "@seedcli/core" {
  interface SeedExtensions {
    my_plugin: {
      greet(name: string): string
    }
  }
}
```

### `tests/plugin.test.ts`
```ts
import { describe, expect, test } from "vitest"
import { build } from "@seedcli/core"
import { createTestCli } from "@seedcli/testing"
import plugin from "../src/index.js"

describe("my-plugin plugin", () => {
  test("hello command runs", async () => {
    const runtime = build("test-cli")
      .plugin(plugin)
      .version("0.1.0")
      .create()

    const result = await createTestCli(runtime).run("hello")

    expect(result.stdout).toContain("Hello, World!")
    expect(result.exitCode).toBe(0)
  })
})
```
