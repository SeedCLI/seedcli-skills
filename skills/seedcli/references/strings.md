# @seedcli/strings
> String manipulation, case conversion, and pluralization.

## Import
```ts
import { camelCase, pascalCase, snakeCase, kebabCase, plural, singular, truncate, template } from "@seedcli/strings"
// Via seed context: seed.strings.*
```

## Case Conversion

```ts
seed.strings.camelCase("hello world")     // "helloWorld"
seed.strings.pascalCase("hello world")    // "HelloWorld"
seed.strings.snakeCase("helloWorld")      // "hello_world"
seed.strings.kebabCase("helloWorld")      // "hello-world"
seed.strings.constantCase("helloWorld")   // "HELLO_WORLD"
seed.strings.titleCase("hello world")     // "Hello World"
seed.strings.sentenceCase("helloWorld")   // "Hello world"
seed.strings.upperFirst("hello")          // "Hello"
seed.strings.lowerFirst("Hello")          // "hello"
```

## Pluralization

```ts
seed.strings.plural("user")        // "users"
seed.strings.plural("person")      // "people"
seed.strings.singular("users")     // "user"
seed.strings.singular("categories") // "category"
seed.strings.isPlural("users")     // true
seed.strings.isSingular("user")    // true
```

## String Manipulation

```ts
seed.strings.truncate("Hello, World!", 8)          // "Hello..."
seed.strings.truncate("Hello, World!", 8, " >>>")  // "Hell >>>"
seed.strings.pad("hi", 10)            // "    hi    "
seed.strings.padStart("42", 5, "0")   // "00042"
seed.strings.padEnd("hi", 10, ".")    // "hi........"
seed.strings.repeat("ab", 3)          // "ababab"
seed.strings.reverse("hello")         // "olleh"
```

## Checks

```ts
seed.strings.isBlank("")           // true (also true for "  ", null, undefined)
seed.strings.isNotBlank("hello")   // true
seed.strings.isEmpty("")           // true (also null, undefined — but NOT "  ")
seed.strings.isNotEmpty("  ")      // true
```

## Simple Templating

```ts
seed.strings.template("Hello, {{name}}! Welcome to {{place}}.", {
  name: "Alice",
  place: "Wonderland",
})
// "Hello, Alice! Welcome to Wonderland."
```
