# Build & Distribution
> Bundle, compile, and distribute Seed CLI projects.

## Import
```ts
import { defineConfig } from "@seedcli/core"
```

## The `seed` CLI Tool

Install: `bun add -g @seedcli/cli` or use `bunx @seedcli/cli <command>`.

## `seed new <name>`

Scaffold a new CLI project:
```bash
seed new my-cli
seed new my-cli --skip-install --skip-git --skip-prompts
```

Templates: Full (recommended), Minimal, or Plugin.

Generated structure (Full):
```
my-cli/
├── src/
│   ├── commands/hello.ts
│   ├── extensions/timer.ts
│   └── index.ts
├── tests/hello.test.ts
├── seed.config.ts
├── package.json
└── tsconfig.json
```

## `seed generate <type> <name>`

Alias: `seed g`.

```bash
seed generate command deploy    # → src/commands/deploy.ts
seed generate extension auth    # → src/extensions/auth.ts
seed generate plugin my-plugin  # → my-plugin/ (full plugin directory)
```

## `seed dev`

Start dev mode with hot reload (`bun --watch`):
```bash
seed dev
seed dev --entry src/cli.ts
```

Entry resolved from: `seed.config.ts` → `package.json` bin → `src/index.ts` → `src/cli.ts` → `index.ts`.

## `seed build`

Bundle or compile for distribution:
```bash
seed build                              # Bundle to dist/
seed build --compile                    # Standalone binary
seed build --compile --target bun-darwin-arm64,bun-linux-x64
seed build --outdir dist --minify --sourcemap
```

### Build Flags

| Flag | Description |
|------|-------------|
| `--compile` | Compile to standalone binary (requires Bun >=1.3.9) |
| `--outfile, -o` | Output file path |
| `--outdir` | Output directory (default: dist) |
| `--target` | Compile targets (comma-separated) |
| `--minify` | Minify output |
| `--sourcemap` | Generate sourcemaps |
| `--bytecode` | Bytecode compilation for faster startup |
| `--splitting` | Enable code splitting |
| `--analyze` | Show bundle size analysis |

### Compile Targets

| Platform | Targets |
|----------|---------|
| macOS | `bun-darwin-arm64`, `bun-darwin-x64` |
| Linux | `bun-linux-x64`, `bun-linux-arm64`, `bun-linux-x64-musl`, `bun-linux-arm64-musl` |
| Windows | `bun-windows-x64`, `bun-windows-arm64` |

The build command auto-converts `.src()` discovery into static imports for bundler compatibility.

## Configuration

```ts
// seed.config.ts
import { defineConfig } from "@seedcli/core"

export default defineConfig({
  dev: {
    entry: "src/index.ts",
  },
  build: {
    entry: "src/index.ts",
    outDir: "dist",
    minify: true,
  },
})
```
