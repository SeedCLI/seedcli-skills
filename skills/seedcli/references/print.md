# @seedcli/print
> Colored logging, spinners, tables, boxes, ASCII art, trees, and progress bars.

## Import
```ts
import { info, success, warning, error, debug, spin, table, box, tree, progressBar, colors } from "@seedcli/print"
// Via seed context: seed.print.*
// Via umbrella: import { info, success, spin, table, box, colors } from "@seedcli/seed"
```

## Logging

```ts
seed.print.info("Starting deployment...")
seed.print.success("Deployed successfully!")
seed.print.warning("Config file not found, using defaults")
seed.print.error("Failed to connect to database")
seed.print.debug("Query took 42ms")  // Only visible with --debug
seed.print.highlight("Important note")
seed.print.muted("Less important info")
seed.print.newline()
seed.print.newline(3)  // 3 blank lines
```

## Colors

Direct `chalk` access:
```ts
const { colors } = seed.print
colors.red("Error!")
colors.bold.green("Success!")
colors.hex("#7cc576")("Custom color")
colors.bgBlue.white(" BADGE ")
```

## Spinner

```ts
const spinner = seed.print.spin("Deploying...")
try {
  await deploy()
  spinner.succeed("Deployed!")
} catch (err) {
  spinner.fail("Deployment failed")
}
```

Methods: `succeed(text?)`, `fail(text?)`, `warn(text?)`, `info(text?)`, `stop()`.
Properties: `text` (update message), `isSpinning`.

## Table

```ts
seed.print.table(rows, {
  headers: ["Package", "Size", "Status"],
  border: "rounded",  // "single" | "double" | "rounded" | "bold" | "none"
  headerColor: colors.cyan,
  maxWidth: 80,
  columns: { 1: { alignment: "right" }, 2: { alignment: "center" } },
})
```

Column config: `alignment` ("left" | "center" | "right"), `width`, `truncate`.

## Box

```ts
seed.print.box("Deployment complete!\nAll services are healthy.", {
  title: "Status",
  borderColor: "green",
  borderStyle: "round",  // "single"|"double"|"round"|"bold"|"singleDouble"|"doubleSingle"|"classic"|"arrow"|"none"
  padding: 1,
  margin: 0,
  textAlignment: "left",  // "left"|"center"|"right"
  float: "left",  // "left"|"center"|"right"
  width: 60,
  fullscreen: false,
})
```

## ASCII Art (Figlet)

```ts
seed.print.ascii("My CLI", { font: "Standard" })
```

Options: `font`, `horizontalLayout`, `verticalLayout`, `width`, `whitespaceBreak`.

## Tree

```ts
seed.print.tree({
  label: "my-project",
  children: [
    { label: "src", children: [{ label: "commands/" }, { label: "index.ts" }] },
    { label: "package.json" },
  ],
})
```

Options: `indent` (default: 2), `guides` (default: true).

## Key-Value

```ts
seed.print.keyValue({ Name: "my-project", Version: "1.0.0", Runtime: "Bun 1.3.5" })
// Or array: seed.print.keyValue([{ key: "Name", value: "my-project" }])
```

Options: `separator` (default: ":"), `keyColor`, `valueColor`, `indent`.

## Divider

```ts
seed.print.divider()
seed.print.divider({ title: "Section", char: "═", color: colors.cyan })
```

Options: `title`, `char` (default: "─"), `width`, `color`, `padding`.

## Progress Bar

```ts
const progress = seed.print.progressBar({
  total: files.length,
  width: 40,
  format: "Processing [:bar] :percent",
  complete: "█",
  incomplete: "░",
})

for (const file of files) {
  await processFile(file)
  progress.update(progress.current + 1)
}
progress.done()
```

## Formatting Helpers

```ts
seed.print.indent("text", 4)       // "    text"
seed.print.wrap(longText, 80)       // Word-wrap at 80 chars
seed.print.columns(items, { columns: 3, padding: 2 })
```
