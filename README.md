# pi-ds4

Pi provider extension for running the model families supported by
[antirez/ds4](https://github.com/antirez/ds4): DeepSeek V4 Flash, DeepSeek V4
Pro, and GLM 5.2. The goal here is to see how good the UX and behavior can be
around local models.

No model is registered or downloaded automatically. Run `/ds4`, choose
**Download model**, and explicitly select a model. Completed downloads are
registered immediately and rediscovered on future Pi starts. Selecting one in
`/model` starts `ds4-server` on demand on a remembered random localhost port.
A per-Pi-process lease and bundled watchdog stop the server when no clients are
left.

## Models

The short model IDs retain the family and quantization:

| Pi model | Family and quant | Resident RAM |
|---|---|---:|
| `ds4/dsv4-flash-q2` | DeepSeek V4 Flash Q2 imatrix | ≥96 GB |
| `ds4/dsv4-flash-q2q4` | DeepSeek V4 Flash mixed Q2/Q4 imatrix | ≥128 GB |
| `ds4/dsv4-flash-q4` | DeepSeek V4 Flash Q4 imatrix | ≥256 GB |
| `ds4/dsv4-pro-q2` | DeepSeek V4 Pro Q2 imatrix | ≥512 GB |
| `ds4/glm52-q4-xl` | GLM 5.2 Unsloth Q4 XL, 11 shards | ≥512 GB |
| `ds4/glm52-iq2xxs` | GLM 5.2 antirez IQ2 XXS | ≥256 GB |
| `ds4/glm52-q2` | GLM 5.2 antirez Q2 | ≥384 GB |
| `ds4/glm52-q4` | GLM 5.2 antirez Q4 | ≥512 GB |

These are conservative full-residency requirements. ds4's SSD-streaming and
distributed modes can run some models with less RAM per machine, at different
performance tradeoffs. The `/ds4` download menu shows the same RAM guidance.

If you are signed into Hugging Face then your token is used for faster downloads.
Package-managed ds4 checkouts are fast-forwarded before use so updates to ds4's
model manifest pull in new uniquely named model releases. Local development
checkouts and explicit `runtimeDir` checkouts are never modified.

## Install

```sh
pi install https://github.com/mitsuhiko/pi-ds4
```

For local development from this checkout, pass the path to an existing ds4 server checkout:

```sh
./install-pi-extension-local.sh /path/to/antirez-ds4-checkout
```

If `~/.pi/ds4/support` already exists and points elsewhere, use `--force` to
move it aside and install a symlink to the checkout you passed. Any existing
`gguf/*.gguf` model files (and resumable `.gguf.part` downloads) are preserved
into the new checkout first, using APFS clone-on-write copies on macOS when
available.

Then restart pi or run `/reload`.

## Runtime layout

Runtime state is kept under `~/.pi/ds4`:

- `support/` — shallow checkout of `https://github.com/antirez/ds4` (`main` by default)
- `kv*` — per-model on-disk KV caches
- `clients/` — active pi process leases
- `port.json` — reserved random local port, guarded by the owning pi/server PID
- `server.json` — live `ds4-server` PID, endpoint, and model state
- `settings.json` — optional extension configuration overrides
- `log` — build/download/server/watchdog log

The watchdog is bundled in this package (`ds4-watchdog.sh`), not expected to
exist in the ds4 runtime checkout.

## Configuration

Environment overrides can also be placed in `~/.pi/ds4/settings.json`.  In the
JSON file, use the env var name (for example `"DS4_READY_TIMEOUT_MS"`), the
camel-case key without `DS4_` (for example `"readyTimeoutMs"`), or the lower
snake-case key without `DS4_` (for example `"ready_timeout_ms"`). Environment
variables win over the settings file.

- `DS4_PROTOCOL`: Pi wire protocol. Supported values are `openai`,
  `openai-responses` (default and recommended), and `anthropic`.
- `DS4_SUPPORT_REPO`: runtime repo URL (default `https://github.com/antirez/ds4`)
- `DS4_SUPPORT_BRANCH`: runtime branch (default `main`)
- `DS4_RUNTIME_DIR`: use an existing ds4 checkout instead of `~/.pi/ds4/support`
- `DS4_CONTEXT_TOKENS`: server and Pi context-window ceiling (default `393216`, the
  minimum context at which ds4 enables DeepSeek V4 Think Max)
- `DS4_AUTO_UPDATE`: fast-forward a package-managed ds4 checkout before use
  (default `true`; never affects local/external checkouts)
- `DS4_READY_TIMEOUT_MS`: server startup timeout
- `DS4_SERVER_BINARY`: custom `ds4-server` binary path
- `DS4_WATCHDOG_SCRIPT`: custom watchdog script path
- `DS4_API_KEY`: provider API key/token sent by Pi (default `dsv4-local`)

See `settings.example.json` for a complete example with a JSON schema reference.
A minimal `~/.pi/ds4/settings.json` can look like this:

```json
{
  "$schema": "https://raw.githubusercontent.com/mitsuhiko/pi-ds4/main/settings.schema.json",
  "protocol": "openai-responses",
  "readyTimeoutMs": 900000
}
```

Use `/ds4` inside Pi to open the management menu:

- **See log** opens the live ds4 log viewer.
- **Start server** or **Stop server** controls the managed server. Starting uses
  the selected ds4 model or asks which installed model to load.
- **Download model** lists all directly runnable DeepSeek V4 Flash, DeepSeek V4
  Pro, and GLM 5.2 variants supported by ds4's downloader. Completed downloads
  are registered immediately and appear in `/model`; they are also rediscovered
  when Pi starts or refreshes its model catalogue.

Distributed DeepSeek V4 Pro Q4 halves are intentionally not Pi models: neither
half is independently runnable without a separately configured multi-host ds4
topology. Optional MTP and DSpark files are acceleration components rather than
standalone models.
