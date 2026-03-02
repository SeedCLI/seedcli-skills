# @seedcli/ui
> Higher-level UI components for rich CLI output.

## Import
```ts
import { header, status, list, countdown } from "@seedcli/ui"
// Via seed context: seed.ui.*
// Via umbrella: import { header, status, uiList, countdown } from "@seedcli/seed"
```

## Header

```ts
seed.ui.header("My CLI", { subtitle: "v1.0.0", color: "green" })
```

Output:
```
╔═══════════════════╗
║      My CLI       ║
║      v1.0.0       ║
╚═══════════════════╝
```

Options: `subtitle`, `color`.

## Status

```ts
seed.ui.status("Database migration", "success")  // ✔
seed.ui.status("Cache invalidation", "fail")      // ✖
seed.ui.status("CDN purge", "skip")               // ○
seed.ui.status("DNS propagation", "pending")      // ◌
```

States: `"success"` (✔), `"fail"` (✖), `"skip"` (○), `"pending"` (◌).

## List

```ts
seed.ui.list(["Install deps", "Run migrations", "Start server"])
seed.ui.list(["First", "Second", "Third"], { ordered: true })
seed.ui.list(["Feature A", "Feature B"], { marker: "arrow" })
```

Options: `ordered` (false), `marker` ("bullet" | "arrow" | "dash" | "number").

## Countdown

```ts
await seed.ui.countdown(5, "Deploying in")
// Deploying in 5... 4... 3... 2... 1...
```

Updates in-place, resolves when reaching zero.

## Re-exported from Print

```ts
seed.ui.divider({ title: "Section" })
seed.ui.keyValue({ Name: "app", Version: "1.0.0" })
seed.ui.tree({ label: "root", children: [{ label: "child" }] })
```

## Example: Deployment Summary

```ts
seed.ui.header("Deployment", { subtitle: "Production", color: "cyan" })
seed.print.newline()
seed.print.keyValue({ Environment: "production", Region: "us-east-1", Version: "2.1.0" })
seed.print.divider("Steps")
seed.ui.status("Build", "success")
seed.ui.status("Test", "success")
seed.ui.status("Deploy", "success")
seed.ui.status("Health check", "pending")
seed.print.box("Deployment complete!", { borderColor: "green", borderStyle: "round", padding: 1 })
```
