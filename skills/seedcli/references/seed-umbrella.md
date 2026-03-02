# @seedcli/seed
> Umbrella package re-exporting all @seedcli/* packages.

## Import
```ts
import { build, command, arg, flag, run, definePlugin, defineExtension, defineConfig } from "@seedcli/seed"
```

## What is @seedcli/seed?

The umbrella package re-exports everything from all `@seedcli/*` packages. Use it when you want a single import source. Some exports are renamed to avoid conflicts:

## Re-export Map

### Core (`@seedcli/core`)
`build`, `Builder`, `Runtime`, `command`, `arg`, `flag`, `run`, `parse`, `route`, `defineConfig`, `definePlugin`, `defineExtension`, `registerModule`, `renderCommandHelp`, `renderGlobalHelp`, `ParseError`

Types: `Seed`, `SeedConfig`, `SeedExtensions`, `Command`, `CommandConfig`, `ArgDef`, `FlagDef`, `Middleware`, `PluginConfig`, `ExtensionConfig`, `ExtensionSeed`, `RunConfig`, `InferArgs`, `InferFlags`

### Print (`@seedcli/print`)
`info`, `success`, `warning`, `error`, `debug`, `highlight`, `muted`, `newline`, `print`, `spin`, `table`, `box`, `ascii` (also as `figlet`), `tree`, `keyValue`, `divider`, `progressBar`, `colors`, `indent`, `wrap`, `columns`, `setDebugMode`

Types: `PrintModule`, `Spinner`, `ProgressBar`, `ProgressBarOptions`, `TableOptions`, `BoxOptions`, `TreeNode`, `TreeOptions`, `DividerOptions`, `KeyValueOptions`, `FigletOptions`, `ColumnConfig`, `ColumnOptions`, `BorderStyle`, `Alignment`, `KeyValuePair`

### Prompt (`@seedcli/prompt`)
`input`, `select`, `multiselect`, `confirm`, `password`, `number`, `editor`, `form`, `autocomplete`, `PromptCancelledError`

Types: `PromptModule`, `InputOptions`, `SelectOptions`, `MultiselectOptions`, `ConfirmOptions`, `PasswordOptions`, `NumberOptions`, `EditorOptions`, `FormField`, `AutocompleteOptions`, `Choice`

### Filesystem (`@seedcli/filesystem`)
`read`, `write`, `readJson`, `writeJson`, `readYaml`, `readToml`, `readBuffer`, `copy`, `move`, `rename`, `remove`, `find`, `exists`, `isFile`, `isDirectory`, `list`, `subdirectories`, `ensureDir`, `stat`, `size`, `tmpDir`, `tmpFile`, `path`, `FileNotFoundError`, `PermissionError`, `DirectoryNotEmptyError`

### System (`@seedcli/system`)
`exec`, `shell`, `which`, `whichOrThrow`, `os`, `arch`, `platform`, `hostname`, `cpus`, `uptime`, `memory`, `env`, `open`, `isInteractive`, `ExecError`, `ExecTimeoutError`, `ExecutableNotFoundError`

### HTTP (`@seedcli/http`) — renamed exports
`httpGet` (get), `httpPost` (post), `httpPut` (put), `httpPatch` (patch), `httpDelete` (delete), `httpHead` (head), `createHttpClient` (create), `createOpenAPIClient`, `download`, `HttpError`, `HttpTimeoutError`

### Config (`@seedcli/config`) — renamed exports
`loadConfig` (load), `loadConfigFile` (loadFile), `getConfig` (get)

### Strings (`@seedcli/strings`)
`camelCase`, `pascalCase`, `snakeCase`, `kebabCase`, `constantCase`, `titleCase`, `sentenceCase`, `upperFirst`, `lowerFirst`, `plural`, `singular`, `isPlural`, `isSingular`, `truncate`, `pad`, `padStart`, `padEnd`, `repeat`, `reverse`, `isBlank`, `isNotBlank`, `isEmpty`, `isNotEmpty`, `template`

### Patching (`@seedcli/patching`) — renamed exports
`patchFile` (patch), `append`, `prepend`, `patternExists` (exists), `patchJson`

### Semver (`@seedcli/semver`) — renamed exports
`valid`, `clean`, `coerce`, `satisfies`, `gt`, `gte`, `lt`, `lte`, `eq`, `compare`, `diff`, `bump`, `major`, `minor`, `patchVersion` (patch), `prerelease`, `sort`, `maxSatisfying`

### Package Manager (`@seedcli/package-manager`) — renamed exports
`detectPackageManager` (detect), `installPackages` (install), `installDevPackages` (installDev), `removePackages` (remove), `runScript` (run), `createPackageManager` (create), `getPackageManagerCommands` (getCommands)

### Template (`@seedcli/template`)
`generate`, `render`, `renderString`, `renderFile`, `directory`

### Testing (`@seedcli/testing`)
`createTestCli`, `mockSeed`, `createInterceptor`

### Completions (`@seedcli/completions`) — renamed exports
`bashCompletions` (bash), `zshCompletions` (zsh), `fishCompletions` (fish), `powershellCompletions` (powershell), `installCompletions` (install), `detectShell` (detect)

### UI (`@seedcli/ui`) — renamed exports
`header`, `status`, `uiList` (list), `countdown`

## When to Use

- **Prototyping** — Single import, no need to remember package names
- **Scripts** — Quick scripts where import convenience matters
- **Full apps** — Prefer direct `@seedcli/*` imports for tree-shaking and clarity
