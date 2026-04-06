# @seedcli/ui
> Higher-level UI components for polished CLI output.

## Import
```ts
import { header, status, list, countdown, divider, keyValue, progress, tree } from "@seedcli/ui"
// Via seed context: seed.ui.*
// Via umbrella: import { header, status, uiList, countdown } from "@seedcli/seed"
```

The high-level helpers return formatted strings or progress helpers. Print them with `seed.print.info(...)` when you want them to appear in command output.

## Header

```ts
seed.print.info(
  seed.ui.header("My CLI", {
    subtitle: "v1.0.0",
    color: "green",
  }),
)
```

Options: `subtitle`, `color`.

## Status

```ts
seed.print.info(seed.ui.status("Database migration", "success"))
seed.print.info(seed.ui.status("Cache invalidation", "fail"))
seed.print.info(seed.ui.status("CDN purge", "skip"))
seed.print.info(seed.ui.status("DNS propagation", "pending"))
```

States:
- `"success"` -> `✔`
- `"fail"` -> `✖`
- `"skip"` -> `◌`
- `"pending"` -> `…`

## List

```ts
seed.print.info(seed.ui.list(["Install dependencies", "Run migrations", "Start server"]))

seed.print.info(
  seed.ui.list(["First step", "Second step", "Third step"], {
    ordered: true,
  }),
)

seed.print.info(
  seed.ui.list(["Feature A", "Feature B"], {
    marker: "arrow",
  }),
)
```

Options: `ordered`, `marker`.

## Countdown

```ts
await seed.ui.countdown(5, "Deploying in")
// Writes "Deploying in 5s", updates in place, then clears the line.
```

## Raw Re-exports

The UI module also re-exports a few lower-level helpers from `@seedcli/print`:

```ts
seed.print.info(seed.ui.divider({ title: "Section" }))
seed.print.info(seed.ui.keyValue({ Name: "app", Version: "1.0.0" }))
seed.print.info(seed.ui.tree({ label: "root", children: [{ label: "child" }] }))

const bar = seed.ui.progress({ total: 3 })
seed.print.info(bar.update(1))
```

## Example: Deployment Summary

```ts
seed.print.info(
  seed.ui.header("Deployment", { subtitle: "Production", color: "cyan" }),
)
seed.print.newline()

seed.print.keyValue({
  Environment: "production",
  Region: "us-east-1",
  Version: "2.1.0",
  Commit: "abc123f",
})
seed.print.newline()

seed.print.divider({ title: "Steps" })
seed.print.info(seed.ui.status("Build", "success"))
seed.print.info(seed.ui.status("Test", "success"))
seed.print.info(seed.ui.status("Deploy", "success"))
seed.print.info(seed.ui.status("Health check", "pending"))
```
