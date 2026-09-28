# OpenJEV support

This fork adds optional [OpenJEV](https://openjev.sh) support alongside the existing TypeSafe and
Cloudflare providers. TypeSafe remains the default; nothing about the existing API changes.

## What was added

| File | Change |
| --- | --- |
| `src/provider.ts` | New `openjev` provider constructor — same wire format as `typesafe`, default base URL `https://api.openjev.sh`. |
| `src/client.ts` | `provider: "openjev"` added to the `ClientOptions` discriminated union; `openjev` in the `defaultModel` map (model id `openjev`); `build()` routes to the `openjev` provider. Key falls back to `OPENJEV_API_KEY`. |
| `test/transport.test.ts` | Test: openjev sends to `https://api.openjev.sh/v1/systemone` with model `openjev`. |
| `README.md` | OpenJEV note after the intro; providers list and docs updated. |
| `README.zh-CN.md` | Same updates in Chinese. |
| `docs/jev-api.md` | OpenJEV transport section under "Other providers". |
| `package.json` | `openjev` keyword added. |

## Provider selection rule

The library uses an explicit `provider` option — there is no auto-detection:

1. `client({ provider: "openjev", apiKey })` → OpenJEV (`OPENJEV_API_KEY` env fallback).
2. `client({ provider: "typesafe", apiKey })` → TypeSafe, unchanged (`TYPESAFE_API_KEY` env fallback).
3. `client({ provider: "cloudflare", accountId, apiKey })` → Cloudflare, unchanged.

Anyone with a TypeSafe key sees zero behaviour change.

## Configuration

Set `OPENJEV_API_KEY` (from https://openjev.sh/dashboard) and choose the provider explicitly:

```ts
import { client } from "@atools/qualm";

const jev = client({ provider: "openjev" }); // reads OPENJEV_API_KEY from the environment
```

Or pass the key directly:

```ts
const jev = client({ provider: "openjev", apiKey: process.env.OPENJEV_API_KEY });
```

## Wire format

OpenJEV uses the same request/response contract as TypeSafe's direct endpoint:

- **Endpoint:** `POST https://api.openjev.sh/v1/systemone`
- **Model:** `openjev`
- **Key:** `OPENJEV_API_KEY` (Bearer token)
- **Request:** `{ model, state, questions }`
- **Response:** `{ model, answers, usage: { input_tokens, output_tokens } }` plus `usage.cost`, `id`, `provider` (extra fields ignored by the library's `WireResult` type).
- **Overload:** HTTP `503` (TypeSafe uses `529`). Both `429` and `5xx` are already retried by `retry.ts`.

## Verification

A live POST to `https://api.openjev.sh/v1/systemone` with model `openjev`, state `ping`, and one
`noul` question returned HTTP 200 with a valid answer. No repository code was executed.

## Upstream

Original project: https://github.com/qddegtya/qualm by @qddegtya (MIT license).
