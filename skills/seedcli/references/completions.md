# @seedcli/completions
> Shell completion generation for bash, zsh, fish, and PowerShell.

## Import
```ts
import { bash, zsh, fish, powershell, install, detect } from "@seedcli/completions"
// Via seed context: seed.completions.*
// Via umbrella: import { bashCompletions, zshCompletions, fishCompletions, powershellCompletions, installCompletions, detectShell } from "@seedcli/seed"
```

## Enabling via Builder

```ts
build("my-cli")
  .completions()  // Adds a "completions" subcommand
  .create()
```

Usage:
```bash
my-cli completions bash        # Output bash completion script
my-cli completions zsh         # Output zsh completion script
my-cli completions fish        # Output fish completion script
my-cli completions powershell  # Output PowerShell completion script
my-cli completions install     # Auto-detect shell and install
```

## Generating Scripts Programmatically

```ts
const info = { brand: "my-cli", commands: [/* ... */] }
const bashScript = bash(info)
const zshScript = zsh(info)
const fishScript = fish(info)
const psScript = powershell(info)
```

## Installing Completions

```ts
await install(info)         // Auto-detect shell
await install(info, "zsh")  // Specific shell
// Returns { shell: "zsh", path: "/path/to/completions" }
```

## Detecting Shell

```ts
const shell = seed.completions.detect()
// "bash" | "zsh" | "fish" | "powershell"
```

## What Gets Completed

Auto-generated from command definitions:
- Command names and aliases
- Subcommand names
- Flag names (`--verbose`, `-v`)
- Flag values (from `choices`)
- Argument values (from `choices`)

## Manual Installation

```bash
my-cli completions bash >> ~/.bashrc
my-cli completions zsh >> ~/.zshrc
my-cli completions fish > ~/.config/fish/completions/my-cli.fish
my-cli completions powershell >> $PROFILE
```
