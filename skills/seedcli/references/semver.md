# @seedcli/semver
> Semantic versioning utilities.

## Import
```ts
import { valid, satisfies, bump, compare, gt, lt, sort, maxSatisfying } from "@seedcli/semver"
// Via seed context: seed.semver.*
```

## Validation & Cleaning

```ts
seed.semver.valid("1.2.3")      // "1.2.3"
seed.semver.valid("not.a.ver")  // null
seed.semver.clean("  v1.2.3  ") // "1.2.3"
seed.semver.coerce("v1")        // "1.0.0"
seed.semver.coerce("3.2")       // "3.2.0"
```

## Comparison

```ts
seed.semver.gt("2.0.0", "1.0.0")      // true
seed.semver.gte("1.0.0", "1.0.0")     // true
seed.semver.lt("1.0.0", "2.0.0")      // true
seed.semver.lte("1.0.0", "1.0.0")     // true
seed.semver.eq("1.0.0", "1.0.0")      // true
seed.semver.compare("1.0.0", "2.0.0") // -1
```

## Range Satisfaction

```ts
seed.semver.satisfies("1.2.3", ">=1.0.0")         // true
seed.semver.satisfies("1.2.3", ">=1.0.0 <2.0.0")  // true
seed.semver.satisfies("2.0.0", "^1.0.0")           // false
```

## Version Parts

```ts
seed.semver.major("1.2.3")             // 1
seed.semver.minor("1.2.3")             // 2
seed.semver.patch("1.2.3")             // 3
seed.semver.prerelease("1.2.3-beta.1") // ["beta", 1]
```

## Bumping

```ts
seed.semver.bump("1.2.3", "major")       // "2.0.0"
seed.semver.bump("1.2.3", "minor")       // "1.3.0"
seed.semver.bump("1.2.3", "patch")       // "1.2.4"
seed.semver.bump("1.2.3", "premajor")    // "2.0.0-0"
seed.semver.bump("1.2.3-0", "prerelease") // "1.2.3-1"
```

## Sorting & Matching

```ts
seed.semver.sort(["3.0.0", "1.0.0", "2.0.0"])  // ["1.0.0", "2.0.0", "3.0.0"]
seed.semver.maxSatisfying(["1.0.0", "1.1.0", "1.2.0", "2.0.0"], "^1.0.0")  // "1.2.0"
```

## Diff

```ts
seed.semver.diff("1.0.0", "2.0.0")  // "major"
seed.semver.diff("1.0.0", "1.1.0")  // "minor"
seed.semver.diff("1.0.0", "1.0.1")  // "patch"
seed.semver.diff("1.0.0", "1.0.0")  // null
```
