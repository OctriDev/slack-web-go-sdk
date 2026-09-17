# Slack Web API Go SDK

One way to interact with the Slack platform is its HTTP RPC-based Web API, a collection of methods requiring OAuth 2.0-based user, bot, or workspace tokens blessed with related OAuth scopes.

> Package `github.com/OctriDev/slack-web-go-sdk` · Version `1.7.0` · 174 operations

## Installation

```sh
# In the consumer module
go mod edit -replace=github.com/OctriDev/slack-web-go-sdk=/path/to/generated-sdk
go get github.com/OctriDev/slack-web-go-sdk

# After this version is published
go get github.com/OctriDev/slack-web-go-sdk@v1.7.0
```

## Quickstart

The argument-free operations in this build use pagination or streaming. Start with the relevant section below so the response is consumed lazily and the connection can be closed correctly.

## Authentication

Keep credentials outside source control. The quickstart reads them from the environment and the client applies them to every request.

| Scheme | ClientAuthConfig field | Sent as |
| --- | --- | --- |
| SDK Studio bearer token | `Bearer` | `Authorization: Bearer <token>` |

## Client behavior

- Base URL: `https://slack.com/api`.
- Transport: net/http.
- Timeout: 30,000 ms per attempt.
- Retries: up to 3 attempts for status codes `408`, `425`, `429`, `500`, `502`, `503`, `504`, with 500–8,000 ms backoff.
- Idempotency: disabled.
- Error telemetry is disabled by default, even when a reporting endpoint is baked into the build. Consumers must opt in explicitly.
- Telemetry PII filtering is enabled by default: common credentials and direct identifiers are recursively replaced with `[REDACTED]` before reports are sent. Disable it only through the generated logging config's `filterPii` (or language-native equivalent) for a trusted private sink.

High-level operation methods return the typed response body directly. The low-level request layer returns an `SdkResponse<T>` envelope containing data, status, headers, request ID, latency, and attempt count.

## Errors and response metadata

All failure paths use a small, predictable hierarchy:

| Error | Meaning |
| --- | --- |
| `SdkValidationError` | A request argument failed an OpenAPI constraint before network I/O. |
| `SdkHttpError` | The server returned a non-2xx response. |
| `SdkNetworkError` | DNS, connection, TLS, or socket failure. |
| `SdkTimeoutError` | The configured per-attempt timeout elapsed. |

HTTP errors expose `StatusCode`, the response body and headers, plus `RequestID` when the server supplies one. Preserve the request ID in support logs; it is the fastest way to correlate a failed SDK call with server-side traces.

## Pagination

1 operation exposes generated pagination helpers. Each operation has a `Paginated` companion; pass a yield callback and return `false` to stop early.

Pagination follows the cursor, offset, page-number, or next-URL contract declared by the OpenAPI operation. It stops when the API signals completion and does not prefetch the entire collection.

## Project layout and API discovery

- Operation implementations are grouped under root-level namespace `.go` files.
- 48 component models are split by API domain under root-level `models_<domain>.go` or `models_<tag_path>_models.go` files in the same public `sdk` package.
- Component schemas can choose a nested model folder with `x-octri-sdk-tags: ["Billing/Invoices"]`; the first tag owns the model and `/` creates nesting.
- [`sdk-manifest.json`](sdk-manifest.json) is the language-neutral public API index: operations, request/response modes, model properties, enum values, and generation settings.
- Public barrel/module exports are the compatibility boundary. Import public model names from those exports; internal domain filenames may evolve without changing model names.

## Links

- [Source repository](https://github.com/OctriDev/slack-web-go-sdk)
- [Issue tracker](https://github.com/OctriDev/slack-web-go-sdk/issues)
- [API documentation](https://api.slack.com/web)
- [Support](https://api.slack.com/support)

<!-- sdk-studio-mock-tests -->
## Local mock-server tests

Generated SDK includes schema-derived, zero-dependency mock server and network
contract suite. Node.js 20+ required. Contract probes use authored response
examples only; schema-synthesized routes remain available to the local server.

`./scripts/mock --port 4010` starts server. `./scripts/test` runs the mock contract suite, then native SDK tests. A zero-authored-example contract run succeeds with an explicit zero-test
summary; mismatches in authored examples still fail.
