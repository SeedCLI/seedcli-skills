# Common Patterns
> Recipes for typical Seed CLI use cases.

## Interactive Init Command

A prompt-driven project initialization that scaffolds files:

```ts
import { command, arg, flag } from "@seedcli/core"

export default command({
  name: "init",
  description: "Initialize a new project",
  args: {
    name: arg({ type: "string", description: "Project name" }),
  },
  flags: {
    template: flag({
      type: "string",
      alias: "t",
      choices: ["minimal", "full", "plugin"] as const,
    }),
    skipInstall: flag({ type: "boolean", alias: "s", description: "Skip dependency installation" }),
  },
  run: async (seed) => {
    // Gather info via prompts (or use flags if provided)
    const name = seed.args.name ?? await seed.prompt.input({ message: "Project name?" })
    const template = seed.flags.template ?? await seed.prompt.select({
      message: "Template?",
      choices: ["minimal", "full", "plugin"] as const,
    })

    const features = await seed.prompt.multiselect({
      message: "Features?",
      choices: ["typescript", "eslint", "prettier", "testing"] as const,
      required: true,
    })

    // Scaffold from template directory
    const spinner = seed.print.spin("Scaffolding project...")
    await seed.template.directory({
      source: `templates/${template}`,
      target: name,
      props: { name, features, hasTypeScript: features.includes("typescript") },
      rename: { "_package.json": "package.json", "_gitignore": ".gitignore" },
    })
    spinner.succeed("Project scaffolded!")

    // Install dependencies
    if (!seed.flags.skipInstall) {
      const installSpinner = seed.print.spin("Installing dependencies...")
      await seed.packageManager.install([], { cwd: name })
      installSpinner.succeed("Dependencies installed!")
    }

    // Initialize git
    await seed.system.exec("git init", { cwd: name })
    await seed.system.exec("git add .", { cwd: name })

    // Summary
    seed.print.newline()
    seed.print.box(`Project "${name}" created!\n\ncd ${name}\n# then run your package manager's dev script`, {
      borderColor: "green",
      borderStyle: "round",
      padding: 1,
    })
  },
})
```

## HTTP API Wrapper Command

Create a typed API client command with retry and error handling:

```ts
import { command, arg, flag } from "@seedcli/core"

export default command({
  name: "users",
  description: "Manage users via API",
  flags: {
    format: flag({ type: "string", choices: ["json", "table"] as const, default: "table" }),
  },
  run: async (seed) => {
    const api = seed.http.create({
      baseURL: seed.system.env("API_URL", "https://api.example.com"),
      headers: { Authorization: `Bearer ${seed.system.env("API_TOKEN")}` },
      retry: { count: 3, backoff: "exponential" },
    })

    const spinner = seed.print.spin("Fetching users...")
    try {
      const { data: users } = await api.get<User[]>("/users")
      spinner.succeed(`Found ${users.length} users`)

      if (seed.flags.format === "json") {
        console.log(JSON.stringify(users, null, 2))
      } else {
        seed.print.table(
          users.map((u) => [u.name, u.email, u.role]),
          { headers: ["Name", "Email", "Role"], border: "rounded" }
        )
      }
    } catch (error) {
      spinner.fail("Failed to fetch users")
      seed.print.error(error.message)
    }
  },
})

interface User {
  name: string
  email: string
  role: string
}
```

## File Scaffolding with Patching

Generate a file from a template and register it in a barrel export:

```ts
import { command, arg } from "@seedcli/core"

export default command({
  name: "generate:model",
  description: "Generate a data model",
  args: {
    name: arg({ type: "string", required: true, description: "Model name" }),
  },
  run: async (seed) => {
    const name = seed.args.name
    const className = seed.strings.pascalCase(name)
    const fileName = seed.strings.kebabCase(name)

    // Generate the model file
    await seed.template.render({
      source: [
        `export interface ${className} {`,
        `  id: string`,
        `  createdAt: Date`,
        `  updatedAt: Date`,
        `}`,
        ``,
        `export function create${className}(data: Partial<${className}>): ${className} {`,
        `  return {`,
        `    id: crypto.randomUUID(),`,
        `    createdAt: new Date(),`,
        `    updatedAt: new Date(),`,
        `    ...data,`,
        `  }`,
        `}`,
      ].join("\n"),
      target: `src/models/${fileName}.ts`,
    })

    // Add to barrel export
    const barrelExists = await seed.filesystem.exists("src/models/index.ts")
    if (barrelExists) {
      await seed.patching.append(
        "src/models/index.ts",
        `export { ${className}, create${className} } from "./${fileName}"\n`
      )
    } else {
      await seed.filesystem.write(
        "src/models/index.ts",
        `export { ${className}, create${className} } from "./${fileName}"\n`
      )
    }

    seed.print.success(`Created model: src/models/${fileName}.ts`)
  },
})
```

## Version Bumping

Semver-aware version management:

```ts
import { command, arg, flag } from "@seedcli/core"

export default command({
  name: "release",
  description: "Bump version and create a release",
  args: {
    type: arg({
      type: "string",
      required: true,
      choices: ["major", "minor", "patch", "premajor", "preminor", "prepatch", "prerelease"] as const,
    }),
  },
  flags: {
    dryRun: flag({ type: "boolean", description: "Preview without making changes" }),
  },
  run: async (seed) => {
    const pkg = await seed.filesystem.readJson<{ version: string }>("package.json")
    const current = pkg.version
    const next = seed.semver.bump(current, seed.args.type)

    if (!next) {
      seed.print.error(`Invalid version bump: ${current} → ${seed.args.type}`)
      return
    }

    seed.print.keyValue({ Current: current, Next: next, Type: seed.args.type })

    if (seed.flags.dryRun) {
      seed.print.muted("Dry run — no changes made")
      return
    }

    const ok = await seed.prompt.confirm({ message: `Bump ${current} → ${next}?` })
    if (!ok) return

    // Update package.json
    await seed.patching.patchJson("package.json", (data) => {
      data.version = next
      return data
    })

    // Git tag and commit
    await seed.system.exec("git add package.json")
    await seed.system.exec(`git commit -m "chore: release v${next}"`)
    await seed.system.exec(`git tag v${next}`)

    seed.print.success(`Released v${next}`)
    seed.print.muted("Run `git push --follow-tags` to publish")
  },
})
```
