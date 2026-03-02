# @seedcli/testing
> Test CLI commands with mocked prompts, config, and system calls.

## Import
```ts
import { createTestCli, mockSeed, createInterceptor } from "@seedcli/testing"
import { describe, test, expect } from "bun:test"
```

## Creating a Test CLI

```ts
const runtime = build("my-cli").src("./src").create()

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

## Mocking Prompts

```ts
const result = await createTestCli(runtime)
  .mockPrompt({
    "Project name?": "my-app",
    "Use TypeScript?": true,
    "License?": "MIT",
  })
  .run("init")
```

## Mocking Config

```ts
const result = await createTestCli(runtime)
  .mockConfig({ outputDir: "test-output", verbose: false })
  .run("build")
```

## Mocking System Commands

```ts
const result = await createTestCli(runtime)
  .mockSystem("git status", { stdout: "On branch main", stderr: "", exitCode: 0 })
  .mockSystem("docker --version", { stdout: "Docker version 24.0.0" })
  .run("deploy")
```

## Chaining Mocks

```ts
const result = await createTestCli(runtime)
  .mockPrompt({ "Continue?": true })
  .mockConfig({ env: "test" })
  .mockSystem("git status", { stdout: "clean", stderr: "", exitCode: 0 })
  .env({ NODE_ENV: "test" })
  .run("deploy staging")
```

## Mock Seed

Unit test command handlers directly:
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

Options: `args`, `flags`, `commandName` ("test"), `brand` ("test-cli"), `version` ("0.0.0").

## Output Interceptor

```ts
const interceptor = createInterceptor()
interceptor.start()

console.log("Hello!")
console.error("Error!")

interceptor.stop()
expect(interceptor.stdout).toContain("Hello!")
expect(interceptor.stderr).toContain("Error!")
```

Note: `createTestCli` uses an interceptor internally.

## Testing Patterns

### Flag validation
```ts
test("rejects invalid port", async () => {
  const result = await createTestCli(runtime).run("serve --port 99999")
  expect(result.stderr).toContain("Port must be between")
  expect(result.exitCode).toBe(1)
})
```

### Interactive flows
```ts
test("init with prompts", async () => {
  const result = await createTestCli(runtime)
    .mockPrompt({
      "Project name?": "test-project",
      "Use TypeScript?": true,
      "Select features:": ["eslint", "prettier"],
    })
    .run("init")
  expect(result.exitCode).toBe(0)
})
```

### Error handling
```ts
test("handles network errors", async () => {
  const result = await createTestCli(runtime)
    .mockSystem("curl https://api.example.com", { stdout: "", stderr: "Connection refused", exitCode: 1 })
    .run("fetch-data")
  expect(result.stderr).toContain("Failed to fetch")
})
```
