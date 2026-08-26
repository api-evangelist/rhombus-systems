---
name: rhombus-sdk-codegen
description: Generate a typed SDK client for the Rhombus API in Python, TypeScript, Java, Go, or C# from the official OpenAPI 3.0 spec. Use whenever the user asks to scaffold a Rhombus client, generate an SDK, create typed API bindings, build a Python/TypeScript/Java Rhombus client, mentions openapi-generator, or says they want typed API calls instead of raw HTTP. Also trigger when the user asks about Rhombus Codegen, the rhombus-api-examples-javascript repo's typed stubs, or how to consume the Rhombus OpenAPI spec programmatically.
argument-hint: "[python|typescript|java|go|csharp] [output-dir]"
---

# Rhombus SDK Codegen

Generate a typed client for the Rhombus API using `openapi-generator-cli` against the live OpenAPI 3.0 spec at `https://api2.rhombussystems.com/api/openapi/public.json`.

## Quick command

```bash
openapi-generator-cli generate \
  -i https://api2.rhombussystems.com/api/openapi/public.json \
  -g <generator> \
  -o <output-dir>
```

Language → generator flag:

| Language | Generator |
|---|---|
| Python | `python` |
| TypeScript (fetch-based) | `typescript-fetch` |
| TypeScript (axios-based) | `typescript-axios` |
| TypeScript (node-based) | `typescript-node` |
| Java | `java` |
| Go | `go` |
| C# | `csharp` |

For the full language × generator × feature matrix, read `references/codegen-matrix.md`.

## Prerequisites

1. **openapi-generator-cli** — install via npm (`npm install -g @openapitools/openapi-generator-cli`) or Homebrew (`brew install openapi-generator`).
2. **Network access** to `api2.rhombussystems.com` so the generator can fetch the live spec.
3. **Language toolchain** for your chosen output (Python 3.8+, Node 18+, JDK 11+, Go 1.20+, .NET 6+).

## After generation

The generated client handles request serialization but not Rhombus-specific auth patterns. Wire up an auth layer:

- **API key flow (server-side):**
  ```python
  client.configuration.api_key['x-auth-apikey'] = os.environ['RHOMBUS_API_KEY']
  client.configuration.api_key['x-auth-scheme'] = 'api-token'
  ```
- **Federated session flow (browser):** Generate a short-lived token via the `/org/generateFederatedSessionToken` endpoint on a server, pass it to the client, and set `x-auth-scheme: federated-session-token`.

See `references/codegen-matrix.md` for language-specific auth helper snippets and known gotchas (e.g., TypeScript-fetch's handling of optional fields, Go's pointer-vs-value generation for nullable types).

## Rhombus Codegen alternative

The `rhombus-api-examples-javascript` repo uses **Rhombus Codegen** (Rhombus's in-house codegen) rather than openapi-generator for TypeScript stubs. If the user is looking at that repo's pattern, clone the repo directly and use its generator; don't try to recreate its output with openapi-generator-cli. Link: `https://github.com/RhombusSystems/rhombus-api-examples-javascript`.

## When to prefer the MCP over a generated SDK

If the user is building an LLM-driven agent or a short script that runs in Claude Code, the `rhombus` MCP (auto-attached by this plugin) is often faster than generating a full SDK. Generate an SDK when:

- The user is shipping a production service (webhooks, backend integrations, long-running daemons).
- The user needs typed IDE autocomplete and compile-time checking.
- The user is integrating into a codebase in a specific language with a conventional dependency management approach.

Use the MCP when the work is exploratory, interactive, or short-lived.
