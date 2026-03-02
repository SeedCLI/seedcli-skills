# @seedcli/system
> Shell commands, system info, and executable discovery.

## Import
```ts
import { exec, shell, which, whichOrThrow, os, arch, env, open, isInteractive } from "@seedcli/system"
// Via seed context: seed.system.*
```

## Command Execution

### `exec(command, options?)`

```ts
const result = await seed.system.exec("git status")
console.log(result.stdout, result.exitCode)

await seed.system.exec("npm install", { stream: true })  // Stream to terminal
await seed.system.exec("echo $MY_VAR", { env: { MY_VAR: "hello" } })
await seed.system.exec("make build", { throwOnError: true })
await seed.system.exec("long-task", { timeout: 30000 })
```

Options: `cwd`, `env`, `stream` (false), `stdin`, `timeout`, `throwOnError` (false), `shell` (true), `trim` (true).
Result: `{ stdout: string, stderr: string, exitCode: number }`.

### `shell`

Bun's `$` shell template literal:
```ts
await seed.system.shell`git add .`
await seed.system.shell`git commit -m "feat: initial"`
const result = await seed.system.shell`cat package.json | grep version`
```

## Executable Discovery

```ts
const dockerPath = seed.system.which("docker")  // string | undefined (sync)
seed.system.whichOrThrow("docker")  // Throws ExecutableNotFoundError if missing (sync)
```

## System Info

```ts
seed.system.os()        // "macos" | "linux" | "windows"
seed.system.arch()      // "x64" | "arm64" | "arm"
seed.system.platform()  // Node.js platform string
seed.system.hostname()
seed.system.cpus()      // Number of CPU cores
seed.system.uptime()    // System uptime in seconds
seed.system.memory()    // { total: number, free: number }
```

## Open

```ts
await seed.system.open("https://seedcli.dev")  // Opens in browser
await seed.system.open("report.pdf")           // Opens in default app
```

## Environment

```ts
seed.system.env("API_TOKEN")         // string | undefined
seed.system.env("PORT", "3000")      // With default
```

## Interactive Detection

```ts
if (seed.system.isInteractive()) {
  const name = await seed.prompt.input({ message: "Name?" })
} else {
  // CI or piped — use defaults
}
```

## Error Types

```ts
import { ExecError, ExecutableNotFoundError, ExecTimeoutError } from "@seedcli/system"
```

| Error | Description |
|-------|-------------|
| `ExecError` | Non-zero exit code (when throwOnError: true) |
| `ExecutableNotFoundError` | Executable not found (from whichOrThrow) |
| `ExecTimeoutError` | Command exceeded timeout |
