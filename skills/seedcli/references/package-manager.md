# @seedcli/package-manager
> Detect and operate with npm, yarn, pnpm, or bun.

## Import
```ts
import {
  create,
  detect,
  detectFromUserAgent,
  getCommands,
  install,
  installDev,
  pmRunPrefix,
  remove,
  run,
} from "@seedcli/package-manager"
```

In Seed commands, `detect`, `create`, `install`, `installDev`, `remove`, `run`, and `getCommands` are also available as `seed.packageManager.*`.

## Detection

```ts
const pm = await seed.packageManager.detect()
// => "bun" | "npm" | "yarn" | "pnpm"
```

Detection priority:
1. `bun.lock` / `bun.lockb`
2. `package-lock.json`
3. `yarn.lock`
4. `pnpm-lock.yaml`
5. `packageManager` field in `package.json`
6. Default: `npm`

## Detect From User Agent

```ts
const pm = detectFromUserAgent()
// => "bun" | "npm" | "yarn" | "pnpm" | undefined
```

Reads `process.env.npm_config_user_agent` to infer the invoking package manager.

## Install Packages

```ts
await seed.packageManager.install(["express", "cors"])
await seed.packageManager.installDev(["typescript", "@types/node"])
await seed.packageManager.install(["express"], {
  cwd: "./my-project",
  exact: true,
  global: false,
  silent: true,
})
```

Options: `cwd`, `exact`, `global`, `silent`.

## Remove Packages

```ts
await seed.packageManager.remove(["express", "cors"])
```

## Run Scripts

```ts
await seed.packageManager.run("build")
await seed.packageManager.run("test", { cwd: "./packages/core" })
await seed.packageManager.run("lint", { args: ["--fix"], silent: true })
```

Options: `cwd`, `args`, `silent`.

## Package Manager Instance

```ts
const pm = await seed.packageManager.create("bun")
// or auto-detect:
const detectedPm = await seed.packageManager.create()

await detectedPm.install(["express"])
await detectedPm.installDev(["typescript"])
await detectedPm.remove(["old-package"])
await detectedPm.run("build")

const version = await detectedPm.version()
console.log(detectedPm.name)
```

## Get Commands

```ts
const commands = seed.packageManager.getCommands("bun")
// {
//   install: "bun install",
//   add: "bun add",
//   addDev: ["bun", "add", "-d"],
//   remove: "bun remove",
//   run: "bun run",
// }
```

## Run Prefix Helper

```ts
pmRunPrefix("npm")  // => "npm run"
pmRunPrefix("pnpm") // => "pnpm run"
pmRunPrefix("bun")  // => "bun run"
pmRunPrefix("yarn") // => "yarn"
```
