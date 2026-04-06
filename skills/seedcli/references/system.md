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
console.log(result.stdout)
console.log(result.exitCode)
```

Options: `cwd`, `env`, `stream`, `stdin`, `timeout`, `throwOnError`, `shell`, `trim`.

Defaults:
- `stream`: `false`
- `throwOnError`: `true`
- `shell`: `true`
- `trim`: `true`

Result: `{ stdout: string, stderr: string, exitCode: number }`.

```ts
await seed.system.exec("npm install", { stream: true })

const envResult = await seed.system.exec('node -p "process.env.MY_VAR"', {
  env: { MY_VAR: "hello" },
})

try {
  await seed.system.exec("make build", { throwOnError: true })
} catch (error) {
  // ExecError with stdout, stderr, exitCode
}
```

### `shell`

`shell` is Execa's `$` template literal, re-exported from `@seedcli/system`:

```ts
const { shell } = seed.system

await shell`git add .`
await shell`git commit -m "feat: initial commit"`

const result = await shell`node -p "process.version"`
console.log(result.stdout)
```

## Executable Discovery

```ts
const dockerPath = seed.system.which("docker")
seed.system.whichOrThrow("docker")
```

Both are synchronous. `whichOrThrow()` throws `ExecutableNotFoundError` if the executable is missing.

## System Info

```ts
seed.system.os()         // "macos" | "linux" | "windows"
seed.system.arch()       // "x64" | "arm64" | "arm"
seed.system.platform()   // Node.js platform string
seed.system.hostname()
seed.system.cpus()
seed.system.uptime()
seed.system.memory()     // { total, free }
```

## Open

```ts
await seed.system.open("https://seedcli.dev")
await seed.system.open("report.pdf")
```

## Environment

```ts
seed.system.env("API_TOKEN")
seed.system.env("PORT", "3000")
```

## Interactive Detection

```ts
if (seed.system.isInteractive()) {
  const name = await seed.prompt.input({ message: "Name?" })
}
```

## Error Types

```ts
import { ExecError, ExecutableNotFoundError, ExecTimeoutError } from "@seedcli/system"
```

| Error | Description |
|-------|-------------|
| `ExecError` | Command exited with a non-zero code by default; set `{ throwOnError: false }` to return an `exitCode` instead |
| `ExecutableNotFoundError` | Executable not found (`whichOrThrow`) |
| `ExecTimeoutError` | Command exceeded timeout |
