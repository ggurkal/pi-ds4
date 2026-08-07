---
name: pi-ds4-config
description: Reconfigure pi-ds4 for local DeepSeek V4 Flash, DeepSeek V4 Pro, and GLM 5.2 models. Use when the user asks how to configure, download models, change protocol/runtime settings, or debug pi-ds4.
---

# pi-ds4 configuration

To reconfigure pi-ds4/ds4, edit `~/.pi/ds4/settings.json` (create it if missing). Do not edit other runtime state files in `~/.pi/ds4` unless explicitly debugging.

The file is a JSON object. Environment variables override it. Keys can be env names (`DS4_READY_TIMEOUT_MS`), camelCase without `DS4_` (`readyTimeoutMs`), or lower snake_case (`ready_timeout_ms`). Use `settings.schema.json` / `settings.example.json` from the pi-ds4 package for validation/examples.

Minimal example:

```json
{
  "$schema": "https://raw.githubusercontent.com/mitsuhiko/pi-ds4/main/settings.schema.json",
  "protocol": "openai-responses",
  "contextTokens": 393216,
  "readyTimeoutMs": 900000
}
```

Common settings:

- `protocol`: `openai`, `openai-responses` (default), or `anthropic`
- `contextTokens`: ds4/Pi context ceiling (default `393216`; GLM models use their lower supported limit)
- `autoUpdate`: fast-forward the package-managed ds4 checkout before use (default `true`; local/external checkouts are untouched)
- `readyTimeoutMs`: server startup timeout in ms
- `runtimeDir`: existing antirez/ds4 checkout instead of `~/.pi/ds4/support`
- `supportRepo` / `supportBranch`: runtime checkout source
- `serverBinary` / `watchdogScript`: custom executable/script paths
- `apiKey`: token Pi sends to the local provider (default `dsv4-local`)

`DS4_GGUF_DIR` is an environment-only upstream ds4 option for storing downloaded GGUF files outside the runtime checkout. It is not a `settings.json` key.

After editing, run `/reload` or restart Pi. Models are never downloaded automatically. Use `/ds4` to download an explicit DeepSeek V4 Flash, DeepSeek V4 Pro, or GLM 5.2 variant and to view the log or control the server. Existing and completed downloads are discovered and registered automatically in Pi's model catalogue.
