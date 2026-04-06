# Build & Distribution
> Bundle Seed CLI apps for Node.js, compile standalone executables, and ship them to users.

## Import
```ts
import { defineConfig } from "@seedcli/core"
```

## The `seed` CLI Tool

Requires **Node.js 24+**.

Install globally:
```bash
npm install -g @seedcli/cli
```

Or use without installing:
```bash
npx @seedcli/cli <command>
```

## `seed new <name>`

Scaffold a new CLI project:
```bash
seed new my-cli
```

Interactive prompts cover:
1. Template — Full, Minimal, or Plugin
2. Description
3. Package manager — `npm`, `pnpm`, `yarn`, or `bun`

Flags:

| Flag | Description |
|------|-------------|
| `--skipInstall, -s` | Skip dependency installation |
| `--skipGit` | Skip git initialization |
| `--skipPrompts, -y` | Skip prompts and use defaults |

Generated structure (Full):
```text
my-cli/
├── src/
│   ├── commands/hello.ts
│   ├── extensions/timer.ts
│   └── index.ts
├── tests/hello.test.ts
├── .gitignore
├── biome.json
├── package.json
├── seed.config.ts
└── tsconfig.json
```

## `seed generate <type> <name>`

Alias: `seed g`.

```bash
seed generate command deploy    # -> src/commands/deploy.ts
seed generate extension auth    # -> src/extensions/auth.ts
seed generate plugin my-plugin  # -> root plugin skeleton + nested plugin/ example package
```

## `seed dev`

Start development mode with automatic restart:
```bash
seed dev
seed dev --entry src/index.ts
```

`seed dev` runs your entry point with `node --watch`. Entry resolution order:
1. `seed.config.ts` -> `dev.entry`
2. `package.json` -> `bin`
3. Common defaults: `src/index.ts`, `src/cli.ts`, `index.ts`

## `seed build`

Bundle for Node.js or compile a standalone executable:
```bash
seed build
seed build --compile
seed build --compile --target node24-macos-arm64,node24-linux-x64,node24-win-x64
seed build --compile --target all
seed build --outdir dist --minify --sourcemap
```

### Build Flags

| Flag | Description |
|------|-------------|
| `--compile` | Compile to a standalone executable via Hakobu |
| `--outfile <path>, -o <path>` | Explicit output file path (single-target only) |
| `--outdir <dir>` | Output directory (default: `dist`) |
| `--target <targets>` | Compile target(s), comma-separated, or `all` |
| `--minify` | Minify output |
| `--sourcemap` | Generate sourcemaps |
| `--external <modules>` | Keep module(s) external (comma-separated) |
| `--splitting` | Parsed by the CLI but currently has no effect |
| `--analyze` | Show bundle size analysis |

### Compile Targets

| Platform | Targets |
|----------|---------|
| macOS | `node24-macos-arm64`, `node24-macos-x64` |
| Linux | `node24-linux-x64`, `node24-linux-arm64`, `node24-linuxstatic-x64` |
| Windows | `node24-win-x64`, `node24-win-arm64` |

`seed build` automatically rewrites dynamic `.src()` discovery into static imports so the bundler/compiler can trace all commands, extensions, and plugins.

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
    bundle: {
      outdir: "dist",
    },
    compile: {
      targets: ["node24-macos-arm64", "node24-linux-x64", "node24-win-x64"],
    },
  },
})
```

## Distributing on npm

Bundle first:
```bash
seed build
```

Typical package configuration:
```json
{
  "type": "module",
  "bin": {
    "my-cli": "./dist/index.js"
  },
  "files": ["dist"],
  "scripts": {
    "build": "seed build",
    "prepublishOnly": "seed build"
  }
}
```

Then publish normally:
```bash
npm publish
```

## Shipping Standalone Binaries

Compile first:
```bash
seed build --compile --target node24-macos-arm64,node24-linux-x64,node24-win-x64 --outdir dist
```

Attach the resulting executables to GitHub Releases, upload them to your own download page, or distribute them through internal artifact storage.

## Signing, Packaging, and Platform Details

Seed uses **Hakobu** as its build backend. Seed handles entry discovery and build defaults; Hakobu handles platform packaging and advanced distribution work.

Use the Hakobu docs for:
- build target details
- bundle mode behavior
- macOS signing and notarization
- Windows signing and metadata
- broader cross-platform distribution guidance

Recommended Hakobu references:
- `https://docs.hakobujs.dev/build/targets`
- `https://docs.hakobujs.dev/build/bundle-mode`
- `https://docs.hakobujs.dev/distribution/cross-platform`
