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

`seed dev` runs your entry point with `node --watch`. For TypeScript entries (`.ts`, `.tsx`, `.mts`, `.cts`), it also loads the [`tsx`](https://tsx.is) ESM loader (`node --watch --import tsx ...`) so the standard TypeScript ESM conventions work:
- `import "./foo.js"` resolves to `foo.ts` when only the TS source exists
- `import "./foo"` (no extension) resolves to `foo.ts`, `foo.tsx`, etc.

`tsx` is included as a dev dependency in scaffolded Full and Minimal projects. For plain `.js`/`.mjs` entries the loader is skipped.

Entry resolution order:
1. `seed.config.ts` -> `dev.entry`
2. `package.json` -> `bin`
3. Common defaults: `src/index.ts`, `src/cli.ts`, `index.ts`

### Forwarding args to your entry script

Anything after a literal `--` is forwarded as `process.argv` to the spawned entry script, and is preserved across hot reloads:
```bash
seed dev -- setup --from /tmp --dryRun
```

The entry script's `process.argv` will receive `["setup", "--from", "/tmp", "--dryRun"]`.

## `seed build`

`seed build` has two modes:

- **Bundle mode** (default) — produces a plain JavaScript ES module at `dist/index.js`, suitable for `npm publish`. Typically tens of KB. Runs directly via `node dist/index.js` (or as a `bin` entry) on Node.js 24+.
- **Compile mode** (`--compile`) — produces a standalone executable via Hakobu with an embedded Node runtime. Tens of MB. Use this when shipping binaries via GitHub Releases.

```bash
seed build                                                          # JS bundle to dist/index.js
seed build --compile                                                # Standalone binary for host
seed build --compile --target node24-macos-arm64,node24-linux-x64,node24-win-x64
seed build --compile --target all
seed build --outdir dist --minify --sourcemap
```

### Bundle Mode Behavior (default)

The bundle is a real JavaScript file (not a Mach-O / PE / ELF binary). `seed build` automatically:

- **Externalizes** `dependencies`, `peerDependencies`, and `optionalDependencies` from `package.json` so the published tarball stays small and uses npm's normal install graph. Workspace packages and `devDependencies` are inlined. Subpaths (e.g. `react/jsx-runtime`) are matched against the package root.
- **Adds a `#!/usr/bin/env node` shebang** unless the entry source already declares one (no double-shebang).
- **`chmod +x`'s** the output so it works directly as a `bin` entry.
- **Emits `dist/index.js.map`** when `--sourcemap` or `build.bundle.sourcemap` is set.

Bundle mode uses **rolldown** under the hood, reused via Hakobu's transitive dep graph (so no second bundler is added to the install tree). Both `seed build` and `seed build --compile` therefore use the same bundler version.

### Build Flags

| Flag | Description |
|------|-------------|
| `--compile` | Compile to a standalone executable via Hakobu (otherwise produce a JS bundle) |
| `--outfile <path>, -o <path>` | Explicit output file path (single-target only) |
| `--outdir <dir>` | Output directory (default: `dist`) |
| `--target <targets>` | Compile target(s), comma-separated, or `all` (used with `--compile`) |
| `--minify` | Minify output |
| `--sourcemap` | Generate sourcemaps |
| `--external <modules>` | Keep additional module(s) external (comma-separated). Works in both modes. |
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
    // Top-level external array — applies to both bundle and compile modes.
    external: ["some-native-module"],
    bundle: {
      outdir: "dist",
      sourcemap: true,
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

`seed build` produces a real ES module at `dist/index.js` (typically tens of KB) with `dependencies`, `peerDependencies`, and `optionalDependencies` from `package.json` kept external. Point `bin` straight at it:

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

The bundle is `chmod +x`'d and gets a `#!/usr/bin/env node` shebang automatically, so it works directly as a `bin` entry.

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
