# @seedcli/template
> Generate files from Eta templates.

## Import
```ts
import { generate, render, renderString, renderFile, directory } from "@seedcli/template"
// Via seed context: seed.template.*
```

## Generate a Single File

```ts
await seed.template.generate({
  template: "component",
  target: "src/components/Button.tsx",
  props: { name: "Button", hasStyles: true },
  directory: "./templates",  // Override templates dir
  overwrite: false,
})
```

Template file `templates/component.tsx.eta`:
```text
import React from 'react'
<% if (it.hasStyles) { %>
import styles from './<%= it.name %>.module.css'
<% } %>

export function <%= it.name %>() {
  return <div><%= it.name %></div>
}
```

## Render to File

```ts
await seed.template.render({
  source: "<%= it.name %> v<%= it.version %>",
  target: "VERSION.txt",
  props: { name: "My App", version: "1.0.0" },
  overwrite: false,
})
```

## Render to String

```ts
const result = await seed.template.renderString("Hello, <%= it.name %>!", { name: "World" })
// "Hello, World!"
```

## Render File to String

```ts
const result = await seed.template.renderFile("templates/readme.md.eta", { projectName: "my-app" })
```

## Scaffold a Directory

```ts
await seed.template.directory({
  source: "templates/project",
  target: "my-new-project",
  props: { name: "my-new-project", useTypeScript: true },
  ignore: ["node_modules/**", ".git/**"],
  rename: { "_package.json": "package.json", "_gitignore": ".gitignore" },
  overwrite: false,
})
```

Files with `.eta` extension are processed as templates (extension stripped). Others copied as-is.

## Template Syntax (Eta)

| Syntax | Description |
|--------|-------------|
| `<%= it.name %>` | Output escaped value |
| `<%~ it.html %>` | Output raw (unescaped) value |
| `<% if (cond) { %> ... <% } %>` | JavaScript logic |
| `<% for (const item of it.items) { %> ... <% } %>` | Loops |
| `<%/* comment */%>` | Comments (not output) |

All variables accessed via the `it` object.
