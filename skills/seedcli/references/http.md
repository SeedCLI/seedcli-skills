# @seedcli/http
> HTTP client with retry support, interceptors, and OpenAPI typed client.

## Import
```ts
import { get, post, put, patch, delete as del, create, createOpenAPIClient, download } from "@seedcli/http"
import { HttpError, HttpTimeoutError } from "@seedcli/http"
// Via seed context: seed.http.*
```

## Simple Requests

```ts
const { data, status } = await seed.http.get<User>("https://api.example.com/users/1")
const { data } = await seed.http.post<User>("https://api.example.com/users", { name: "Alice" })
await seed.http.put("https://api.example.com/users/1", { name: "Alice Updated" })
await seed.http.patch("https://api.example.com/users/1", { name: "Alice" })
await seed.http.delete("https://api.example.com/users/1")
const { headers } = await seed.http.head("https://api.example.com/health")
```

Request options: `headers`, `params` (query), `timeout`, `retry`, `signal`.
Response: `{ data: T, status, statusText, headers, ok, raw }`.

## Retry Configuration

```ts
await seed.http.get(url, {
  retry: { count: 3, delay: 1000, backoff: "exponential", retryOn: [503, 429], onRetry: (err, attempt) => {} },
})
```

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `count` | `number` | — | Number of retries |
| `delay` | `number` | 1000 | Initial delay in ms |
| `backoff` | `"linear" \| "exponential"` | "linear" | Backoff strategy |
| `retryOn` | `number[]` | [408,429,500,502,503,504] | Status codes to retry |
| `onRetry` | `(error, attempt) => void` | — | Callback per retry |

## HTTP Client Factory

```ts
const api = seed.http.create({
  baseURL: "https://api.example.com",
  headers: { Authorization: `Bearer ${token}` },
  timeout: 10000,
  retry: 2,
  interceptors: {
    request: async (url, init) => { /* modify init */ return init },
    response: async (response) => { /* inspect response */ return response },
  },
})

const { data } = await api.get<User[]>("/users")
const { data } = await api.post<User>("/users", { name: "Alice" })
```

## File Download

```ts
await seed.http.download("https://example.com/archive.zip", "./downloads/archive.zip", {
  onProgress: (p) => console.log(`${p.percent}% - ${p.speed} bytes/s`),
})
```

Progress: `{ percent, transferred, total, speed }`.

## OpenAPI Typed Client

```ts
import type { paths } from "./api-schema"

const api = seed.http.createOpenAPIClient<paths>({
  baseUrl: "https://api.example.com",
  headers: { Authorization: `Bearer ${token}` },
})

const { data, error } = await api.GET("/users/{id}", { params: { path: { id: "123" } } })
const { data } = await api.POST("/users", { body: { name: "Alice" } })
```

Generate types: `bunx openapi-typescript https://api.example.com/openapi.json -o src/api-schema.ts`

## Error Types

| Error | Description |
|-------|-------------|
| `HttpError` | Non-2xx response (includes status, statusText, data) |
| `HttpTimeoutError` | Request exceeded timeout |
