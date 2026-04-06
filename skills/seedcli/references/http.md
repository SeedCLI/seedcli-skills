# @seedcli/http
> HTTP client with retry support and an optional OpenAPI typed client.

## Import
```ts
import {
  create,
  createOpenAPIClient,
  delete as del,
  download,
  get,
  head,
  patch,
  post,
  put,
} from "@seedcli/http"
import { HttpError, HttpTimeoutError } from "@seedcli/http"
// Via seed context: seed.http.*
```

## Simple Requests

```ts
const { data, status } = await seed.http.get<User>("https://api.example.com/users/1")
const { data } = await seed.http.post<User>("https://api.example.com/users", {
  name: "Alice",
  email: "alice@example.com",
})
await seed.http.put("https://api.example.com/users/1", { name: "Alice Updated" })
await seed.http.patch("https://api.example.com/users/1", { name: "Alice" })
await seed.http.delete("https://api.example.com/users/1")
const { headers } = await seed.http.head("https://api.example.com/health")
```

Request options: `headers`, `params`, `timeout`, `retry`, `signal`.

## Retry Configuration

```ts
await seed.http.get("https://api.example.com/data", {
  retry: {
    count: 3,
    delay: 1000,
    backoff: "exponential",
    retryOn: [503, 429],
  },
})
```

Defaults:
- `delay`: `1000`
- `backoff`: `"exponential"`
- `retryOn`: `[408, 429, 500, 502, 503, 504]`

## HTTP Client Factory

```ts
const api = seed.http.create({
  baseURL: "https://api.example.com",
  headers: {
    Authorization: `Bearer ${token}`,
    "Content-Type": "application/json",
  },
  timeout: 10000,
  retry: 2,
})

const { data: users } = await api.get<User[]>("/users")
const { data: user } = await api.post<User>("/users", { name: "Alice" })
```

Client config supports `baseURL`, `headers`, `timeout`, `retry`, and request/response `interceptors`.

## File Download

```ts
await seed.http.download("https://example.com/archive.zip", "./downloads/archive.zip", {
  onProgress: (progress) => {
    console.log(`${progress.percent}% - ${progress.speed} bytes/s`)
  },
})
```

Progress shape: `{ percent, transferred, total, speed }`.

## OpenAPI Typed Client

```ts
import { createOpenAPIClient } from "@seedcli/http"
import type { paths } from "./api-schema"

const api = await createOpenAPIClient<paths>({
  baseUrl: "https://api.example.com",
  headers: { Authorization: `Bearer ${token}` },
})

const { data, error } = await api.GET("/users/{id}", {
  params: { path: { id: "123" } },
})
```

If you use `createOpenAPIClient()`, also install the optional peer dependency:

```bash
npm install openapi-fetch
```

Generate types from a schema:
```bash
npx openapi-typescript https://api.example.com/openapi.json -o src/api-schema.ts
```

## Error Types

| Error | Description |
|-------|-------------|
| `HttpError` | Non-2xx response (includes `status`, `statusText`, `data`) |
| `HttpTimeoutError` | Request exceeded timeout |
