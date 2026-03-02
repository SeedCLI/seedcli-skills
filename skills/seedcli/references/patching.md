# @seedcli/patching
> Modify existing files with surgical precision.

## Import
```ts
import { patch, append, prepend, exists, patchJson } from "@seedcli/patching"
// Via seed context: seed.patching.*
// Via umbrella: import { patchFile, append, prepend, patternExists, patchJson } from "@seedcli/seed"
```

## Patch a File

```ts
const result = await seed.patching.patch("src/app.ts", {
  insert: "import { logger } from './logger'\n",
  before: "import { router }",
})
// result: { changed: boolean, content: string }
```

### Patch Options

| Option | Type | Description |
|--------|------|-------------|
| `insert` | `string` | Content to insert (with before/after/replace) |
| `before` | `string \| RegExp` | Insert before this pattern |
| `after` | `string \| RegExp` | Insert after this pattern |
| `replace` | `string \| RegExp` | Replace this pattern (use insert for replacement text) |
| `delete` | `string \| RegExp` | Delete this pattern |

### Examples

```ts
// Insert before
await seed.patching.patch("src/routes.ts", {
  insert: "  app.use('/api/users', usersRouter)\n",
  before: "// END ROUTES",
})

// Insert after
await seed.patching.patch("src/index.ts", {
  insert: "import { analytics } from './analytics'\n",
  after: "import { app } from './app'\n",
})

// Replace
await seed.patching.patch("config.json", {
  replace: /"version": "\d+\.\d+\.\d+"/,
  insert: '"version": "2.0.0"',
})

// Delete
await seed.patching.patch("src/app.ts", {
  delete: /import { oldModule } from '\.\/old-module'\n/,
})
```

## Append & Prepend

```ts
await seed.patching.append("src/styles.css", "\n.new-class { color: red; }\n")
await seed.patching.prepend("src/app.ts", "// Generated file — do not edit\n")
```

## Check for Pattern

```ts
const hasImport = await seed.patching.exists("src/app.ts", "import { logger }")
const hasRoute = await seed.patching.exists("src/routes.ts", /app\.use\(['"]\/api\/users/)
```

## Patch JSON

```ts
await seed.patching.patchJson("package.json", (data) => {
  data.scripts ??= {}
  data.scripts.lint = "biome check"
  return data
})

await seed.patching.patchJson<PkgJson>("package.json", (data) => {
  data.dependencies["new-dep"] = "^1.0.0"
  return data
})
```

## Common Patterns

### Add route + import
```ts
const exists = await seed.patching.exists("src/routes.ts", `app.use('/api/${name}'`)
if (!exists) {
  await seed.patching.patch("src/routes.ts", {
    insert: `import { ${name}Router } from './routes/${name}'\n`,
    after: /import .* from '\.\/routes\/.*'\n/,
  })
  await seed.patching.patch("src/routes.ts", {
    insert: `  app.use('/api/${name}', ${name}Router)\n`,
    before: "// END ROUTES",
  })
}
```

### Add barrel export
```ts
await seed.patching.append("src/index.ts", `export { ${name} } from './${name}'\n`)
```
