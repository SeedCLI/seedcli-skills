# @seedcli/prompt
> Interactive prompts with full type inference.

## Import
```ts
import { input, select, multiselect, confirm, password, number, editor, form, autocomplete, PromptCancelledError } from "@seedcli/prompt"
// Via seed context: seed.prompt.*
```

## Input

```ts
const name = await seed.prompt.input({ message: "What is your name?" })
const email = await seed.prompt.input({
  message: "Email?",
  default: "user@example.com",
  validate: (v) => v.includes("@") || "Must be a valid email",
  transformer: (v) => v.toLowerCase().trim(),
})
```

Options: `message` (required), `default`, `required`, `validate`, `transformer`.

## Number

```ts
const port = await seed.prompt.number({ message: "Port?", default: 3000, min: 1, max: 65535 })
```

Options: `message`, `default`, `min`, `max`, `step`, `required`, `validate`.

## Confirm

```ts
const ok = await seed.prompt.confirm({ message: "Deploy to production?", default: false })
```

## Password

```ts
const token = await seed.prompt.password({
  message: "API token?",
  mask: "*",
  validate: (v) => v.length >= 8 || "Token must be at least 8 characters",
})
```

## Select

```ts
// Simple choices (type-narrowed with as const)
const env = await seed.prompt.select({
  message: "Target environment?",
  choices: ["development", "staging", "production"] as const,
})
// env: "development" | "staging" | "production"

// Labeled choices
const db = await seed.prompt.select({
  message: "Database?",
  choices: [
    { name: "PostgreSQL", value: "postgres" },
    { name: "MySQL", value: "mysql" },
    { name: "SQLite", value: "sqlite", description: "File-based" },
  ] as const,
})
```

Options: `message`, `choices` (required), `default`, `loop`.

## Multiselect

```ts
const features = await seed.prompt.multiselect({
  message: "Select features:",
  choices: ["typescript", "eslint", "prettier", "testing"] as const,
  required: true,
})
// features: ReadonlyArray<"typescript" | "eslint" | "prettier" | "testing">
```

Options: `message`, `choices`, `default`, `required`, `loop`, `validate`.

## Autocomplete

```ts
const pkg = await seed.prompt.autocomplete({
  message: "Search packages:",
  source: async (input) => {
    if (!input) return []
    const res = await fetch(`https://registry.npmjs.org/-/v1/search?text=${input}`)
    const data = await res.json()
    return data.objects.map((o: any) => o.package.name)
  },
})
```

## Editor

```ts
const message = await seed.prompt.editor({
  message: "Commit message:",
  default: "feat: ",
  postfix: ".md",
})
```

Options: `message`, `default`, `postfix`, `validate`, `waitForUserInput`.

## Form

```ts
const answers = await seed.prompt.form<{ name: string; version: string; private: boolean }>([
  { name: "name", type: "input", message: "Project name?" },
  { name: "version", type: "input", message: "Version?", default: "0.0.1" },
  { name: "private", type: "confirm", message: "Private?", default: true },
  { name: "license", type: "select", message: "License?", choices: ["MIT", "Apache-2.0"] },
])
```

Field types: `"input"`, `"number"`, `"confirm"`, `"password"`, `"select"`.

## Cancellation

All prompts throw `PromptCancelledError` on Ctrl+C:
```ts
import { PromptCancelledError } from "@seedcli/prompt"

try {
  const name = await seed.prompt.input({ message: "Name?" })
} catch (error) {
  if (error instanceof PromptCancelledError) {
    seed.print.info("Cancelled.")
    return
  }
  throw error
}
```
