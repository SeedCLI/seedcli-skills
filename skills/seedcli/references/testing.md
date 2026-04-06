# @seedcli/testing
> Test CLI commands by capturing stdout/stderr, exit codes, and temporary environment variables.

## Import
```ts
import { createTestCli, mockSeed, createInterceptor } from "@seedcli/testing"
import { describe, expect, test } from "vitest"
```

## Creating a Test CLI

```ts
import { build } from "@seedcli/core"

const runtime = build("my-cli")
  .src("./src")
  .create()

describe("greet command", () => {
  test("greets the user", async () => {
    const result = await createTestCli(runtime).run("greet Alice")
    expect(result.stdout).toContain("Hello, Alice!")
    expect(result.exitCode).toBe(0)
  })

  test("shows error for missing name", async () => {
    const result = await createTestCli(runtime).run("greet")
    expect(result.stderr).toContain("required")
    expect(result.exitCode).toBe(1)
  })
})
```

Result: `{ stdout: string, stderr: string, exitCode: number }`.

## Builder Methods

`createTestCli(runtime)` returns a chainable builder with:
- `mockPrompt(responses)`
- `mockConfig(config)`
- `mockSystem(command, result)`
- `env(vars)`
- `run(argv)`

In the current implementation, only `env()` affects `run()`. The `mockPrompt()`, `mockConfig()`, and `mockSystem()` helpers are exposed on the builder but are not applied during execution yet.

```ts
const result = await createTestCli(runtime)
  .env({ NODE_ENV: "test" })
  .run("deploy staging")
```

## Mock Seed

Unit-test command handlers directly:
```ts
const seed = mockSeed({
  args: { environment: "staging" },
  flags: { force: true },
  commandName: "deploy",
  brand: "my-cli",
  version: "1.0.0",
})

await deployCommand.run(seed)
```

Options: `args`, `flags`, `commandName`, `brand`, `version`.

All module methods on the mock are no-ops by default, so override only the pieces your handler needs.

## Output Interceptor

```ts
const interceptor = createInterceptor()
interceptor.start()

console.log("Hello!")
console.error("Error!")

interceptor.stop()
expect(interceptor.stdout).toContain("Hello!")
expect(interceptor.stderr).toContain("Error!")
expect(interceptor.exitCode).toBe(0)
```

`createTestCli()` uses an interceptor internally, so you only need `createInterceptor()` when testing output manually.

## Testing Patterns

### Flag validation
```ts
test("rejects invalid port", async () => {
  const result = await createTestCli(runtime).run("serve --port 99999")
  expect(result.stderr).toContain("Port must be between")
  expect(result.exitCode).toBe(1)
})
```

### Environment-sensitive behavior
```ts
test("uses CI defaults in non-interactive mode", async () => {
  const result = await createTestCli(runtime)
    .env({ CI: "1" })
    .run("init")

  expect(result.exitCode).toBe(0)
})
```
