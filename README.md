# pi-ds4

Run [antirez/ds4](https://github.com/antirez/ds4) models locally in Pi. The
extension supports DeepSeek V4 Flash, DeepSeek V4 Pro, and GLM 5.2.

## Install

```sh
pi install https://github.com/mitsuhiko/pi-ds4
```

Restart Pi or run `/reload`.

## Get started

1. Run `/ds4` and choose **Download model**.
2. Pick a model. DeepSeek V4 Flash Q2 is the smallest option and needs about
   96 GB of RAM.
3. After the download finishes, run `/model` and select the new `ds4/...`
   model.
4. Start chatting.

Models are never downloaded automatically. Once installed, they are discovered
on future Pi starts and shown in `/model`. Selecting one starts `ds4-server` on
demand; it stops automatically when no Pi processes are using it.

Use `/ds4` at any time to download another model, view the log, or start and
stop the server manually.

## Models

| Pi model | Model | Recommended RAM |
|---|---|---:|
| `ds4/dsv4-flash-q2` | DeepSeek V4 Flash Q2 imatrix | 96 GB |
| `ds4/dsv4-flash-q2q4` | DeepSeek V4 Flash mixed Q2/Q4 imatrix | 128 GB |
| `ds4/dsv4-flash-q4` | DeepSeek V4 Flash Q4 imatrix | 256 GB |
| `ds4/dsv4-pro-q2` | DeepSeek V4 Pro Q2 imatrix | 512 GB |
| `ds4/glm52-iq2xxs` | GLM 5.2 IQ2 XXS | 256 GB |
| `ds4/glm52-q2` | GLM 5.2 Q2 | 384 GB |
| `ds4/glm52-q4` | GLM 5.2 Q4 | 512 GB |
| `ds4/glm52-q4-xl` | GLM 5.2 Unsloth Q4 XL | 512 GB |

These are conservative full-residency recommendations. ds4 can also stream
from SSD or run across multiple machines, with different performance tradeoffs.
Signing in to Hugging Face before downloading may improve download speed.

## Configuration

Most users do not need any configuration. Optional settings live in
`~/.pi/ds4/settings.json`:

```json
{
  "$schema": "https://raw.githubusercontent.com/mitsuhiko/pi-ds4/main/settings.schema.json",
  "protocol": "openai-responses",
  "readyTimeoutMs": 900000
}
```

See [`settings.example.json`](settings.example.json) for all options. Each
setting also has a `DS4_*` environment variable; environment variables take
precedence. Restart Pi or run `/reload` after changing settings.

Runtime files, downloaded models, caches, and logs are stored in `~/.pi/ds4`.
Package-managed ds4 checkouts update automatically; explicitly configured or
local development checkouts are left untouched.

## Local development

To use this extension checkout with an existing ds4 checkout:

```sh
./install-pi-extension-local.sh /path/to/antirez-ds4-checkout
```

If `~/.pi/ds4/support` already points elsewhere, rerun with `--force`. Existing
model downloads are preserved when possible.
