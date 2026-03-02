# @seedcli/filesystem
> File operations, path helpers, and directory management.

## Import
```ts
import { read, write, readJson, writeJson, copy, move, find, exists, path, tmpDir } from "@seedcli/filesystem"
// Via seed context: seed.filesystem.*
```

## Reading Files

```ts
const content = await seed.filesystem.read("config.json")
const pkg = await seed.filesystem.readJson<{ name: string }>("package.json")
const config = await seed.filesystem.readYaml("config.yaml")
const cargo = await seed.filesystem.readToml("Cargo.toml")
const buffer = await seed.filesystem.readBuffer("image.png")
```

## Writing Files

```ts
await seed.filesystem.write("output.txt", "Hello, world!")
await seed.filesystem.writeJson("config.json", { name: "my-app" }, { indent: 2, sortKeys: true })
```

## Copy & Move

```ts
await seed.filesystem.copy("src/template.ts", "dist/template.ts")
await seed.filesystem.copy("src", "dist", { overwrite: true, filter: (p) => !p.includes("node_modules") })
await seed.filesystem.move("old.ts", "new.ts")
await seed.filesystem.move("file.ts", "archive/file.ts", { overwrite: true })
await seed.filesystem.rename("old.ts", "new.ts")
```

## Remove

```ts
await seed.filesystem.remove("dist")  // Recursive
```

## Finding Files

```ts
const tsFiles = await seed.filesystem.find("src", { matching: "**/*.ts" })
const assets = await seed.filesystem.find(".", {
  matching: ["**/*.png", "**/*.jpg"],
  ignore: ["node_modules/**"],
})
const dirs = await seed.filesystem.find(".", { directories: true, files: false, matching: "packages/*" })
```

Options: `matching`, `ignore`, `files` (true), `directories` (false), `recursive` (true), `dot` (false).

## Existence Checks

```ts
await seed.filesystem.exists("config.json")    // boolean
await seed.filesystem.isFile("config.json")     // boolean
await seed.filesystem.isDirectory("src")        // boolean
```

## Path Helpers

```ts
const { path } = seed.filesystem
path.resolve("src", "commands")
path.join("src", "commands", "deploy.ts")
path.dirname("/app/src/cli.ts")     // "/app/src"
path.basename("/app/src/cli.ts")    // "cli.ts"
path.ext("/app/src/cli.ts")         // ".ts"
path.isAbsolute("/app/src")         // true
path.relative("/app", "/app/src")   // "src"
path.home()                          // User home directory
path.cwd()                           // Current working directory
```

## Directory Operations

```ts
const files = await seed.filesystem.list("src/commands")  // ["deploy.ts", "init.ts"]
const dirs = await seed.filesystem.subdirectories("packages")
await seed.filesystem.ensureDir("output/reports")  // Creates if missing
```

## Temp Files & Directories

```ts
const tmpDir = await seed.filesystem.tmpDir({ prefix: "build-" })
const tmpFile = await seed.filesystem.tmpFile({ ext: ".json", prefix: "config-" })
```

## File Info

```ts
const info = await seed.filesystem.stat("package.json")
// { size, created, modified, accessed, isFile, isDirectory, isSymlink, permissions }
const bytes = await seed.filesystem.size("bundle.js")
```

## Error Types

```ts
import { FileNotFoundError, PermissionError, DirectoryNotEmptyError } from "@seedcli/filesystem"
```
