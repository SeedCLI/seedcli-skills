# @seedcli/package-manager
> Detect and operate with npm, yarn, pnpm, or bun.

## Import
```ts
import { detect, install, installDev, remove, run, create } from "@seedcli/package-manager"
// Via seed context: seed.packageManager.*
// Via umbrella: import { detectPackageManager, installPackages, installDevPackages, removePackages, runScript, createPackageManager } from "@seedcli/seed"
```

## Detection

```ts
const pm = await seed.packageManager.detect()
// "bun" | "npm" | "yarn" | "pnpm"
```

Priority: `bun.lock`/`bun.lockb` → `package-lock.json` → `yarn.lock` → `pnpm-lock.yaml` → `packageManager` field → default: `bun`.

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

Options: `cwd`, `exact` (false), `global` (false), `silent` (false).

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
const pm = await seed.packageManager.create()      // Auto-detect
const pm = await seed.packageManager.create("bun")  // Explicit

await pm.install(["express"])
await pm.installDev(["typescript"])
await pm.remove(["old-package"])
await pm.run("build")
const version = await pm.version()  // "1.3.5"
console.log(pm.name)                // "bun"
```

## Get Commands

```ts
seed.packageManager.getCommands("bun")
// { install: "bun install", add: "bun add", addDev: ["bun", "add", "-d"], remove: "bun remove", run: "bun run" }
```
