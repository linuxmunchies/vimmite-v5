# Configuring Strix Halo local AI

This guide explains how to change the behavior of the host commands installed
by `ujust strix-halo-ai`. You do not need to enter a Distrobox for ordinary
use. Start with a one-time command-line change; make a preset only when you
want settings to persist across server starts.

## The two normal launch styles

With no model argument, both server commands start a localhost router. It
discovers the model bundles below `~/ai/models` and loads the requested model
only when a client asks for it:

```bash
llama-server-rocm
# or
llama-server-vulkan
```

List the model IDs that a client should use:

```bash
curl -fsS http://127.0.0.1:8080/v1/models | jq -r '.data[] | [.id, .status.value] | @tsv'
```

To run one model directly instead of using the router, pass `-m` (or
`--model`). This is useful for experiments and automatically preserves the
same Strix Halo GPU defaults:

```bash
llama-server-rocm \
  -m "$HOME/ai/models/qwen3.6-35b-a3b/Qwen3.6-35B-A3B-UD-Q4_K_XL.gguf" \
  -c 8192 --port 9000
```

## One-time server changes

Put options after either `llama-server-*` command. They apply only to that
server process and take precedence over the setup defaults.

| What to change | Example | Effect |
| --- | --- | --- |
| Context size | `-c 8192` | Maximum context per model/slot. Start modestly; larger contexts consume more unified memory. |
| Parallel request slots | `-np 2` | Lets a model serve two concurrent requests. Each slot needs its own context/KV memory. |
| Loaded router models | `--models-max 2` | Keeps at most two model instances resident at once. The default is one. |
| Server port | `--port 9000` | Uses a non-default local port. |
| LAN binding | `--host 0.0.0.0 --port 9000` | Deliberately exposes the server beyond localhost. Review your firewall and authentication first. |
| GPU layers | `-ngl 999` | Default is effectively all layers on the GPU. `-ngl 0` is a CPU-only diagnostic. |
| Flash Attention | `--flash-attn off` | Disables the default enabled Flash Attention. |
| Model loading | `--load-mode mmap` | Replaces the default `--load-mode none`. |
| Qwen MTP length | `--spec-draft-n-max 2` | Changes the default Qwen draft length from three tokens to two. |
| Disable MTP | `--spec-type none` | Disables speculative decoding for this launch. |

Examples:

```bash
# A conservative router: 8K context, one concurrent request, one loaded model.
llama-server-rocm -c 8192 -np 1 --models-max 1

# Keep the defaults except use a different port.
llama-server-vulkan --port 9000

# Test a Qwen model without MTP.
llama-server-vulkan --spec-type none
```

The normal defaults are `--load-mode none -ngl 999 -fa 1`, plus
`--host 127.0.0.1 --port 8080` for servers. Supplying an equivalent option
prevents the wrapper from adding its default. The older spelling `--no-mmap`
still works as an explicit override, but llama.cpp currently recommends
`--load-mode` instead.

## Persistent per-model settings

The setup manages this file:

```text
~/.local/share/strix-halo-ai/router-presets.ini
```

Do **not** edit it for your own configuration: installation, update, or wrapper
refresh can replace it. Make a user-owned copy instead:

```bash
mkdir -p "$HOME/.config/strix-halo-ai"
cp "$HOME/.local/share/strix-halo-ai/router-presets.ini" \
  "$HOME/.config/strix-halo-ai/my-models.ini"
```

Edit `~/.config/strix-halo-ai/my-models.ini` in a text editor. For example:

```ini
version = 1

; Settings shared by every routed model unless a model section overrides them.
[*]
ctx-size = 8192
parallel = 1

; Qwen 3.6: retain MTP but draft two tokens at a time.
[qwen3.6-35b-a3b]
spec-type = draft-mtp
spec-draft-n-max = 2

; Qwen 3.8: turn MTP off for this model only.
[qwen3.8-27b]
spec-type = none
```

Use the custom file explicitly:

```bash
llama-server-rocm \
  --models-dir "$HOME/ai/models" \
  --models-preset "$HOME/.config/strix-halo-ai/my-models.ini"
```

The same command works with `llama-server-vulkan`. Stop the current server
with `Ctrl+C`, then start it again after editing a preset.

`[*]` is for settings shared by model instances; named sections use the router
model IDs (`muse-glimmer-30b`, `qwen3.6-35b-a3b`, and `qwen3.8-27b`).
Command-line options have higher priority than preset entries.

The wrapper itself supplies `--load-mode`, `-ngl`, and `-fa` on the command
line. Therefore change those three settings with a server launch option, not
inside a model preset. For example, `llama-server-rocm --flash-attn off` is a
whole-server change, while `ctx-size` and the Qwen MTP settings can differ per
model in a preset.

## Reusing your custom preset conveniently

If you always want the custom preset, set this variable before launching a
server:

```bash
export STRIX_HALO_AI_ROUTER_PRESETS="$HOME/.config/strix-halo-ai/my-models.ini"
llama-server-rocm
```

Add the `export` line to `~/.zshrc` only if you want that behavior in every new
terminal. The wrapper will then keep its convenient no-argument router mode
while using your user-owned preset.

Other useful temporary overrides are:

```bash
# Point the default router at a different model directory.
STRIX_HALO_AI_MODEL_ROOT="/path/to/models" llama-server-rocm

# Print, rather than run, the Distrobox command the wrapper would use.
STRIX_HALO_AI_DRY_RUN=1 llama-server-vulkan -c 8192
```

When using a different model directory, organize each model's GGUF files in
its own subdirectory, as the managed `~/ai/models` layout does. A multimodal
projector stays alongside its matching model file.

## Changing a client configuration

The default local OpenAI-compatible endpoint is:

```text
http://127.0.0.1:8080/v1
```

For Pi Agent or another OpenAI-compatible client, set the model field to the
router ID, for example `qwen3.6-35b-a3b`, rather than the full GGUF path. If
you change `--port`, update the client's base URL to match. A client that needs
an API key can use any non-empty placeholder while the server remains bound to
localhost; configure a real API key before intentionally exposing a server to
your LAN.

## If something goes wrong

```bash
# Check that the server is listening.
ss -ltnp 'sport = :8080'

# Check router-discovered models and their load state.
curl -fsS http://127.0.0.1:8080/v1/models | jq

# Test the server itself.
curl -fsS http://127.0.0.1:8080/health

# Confirm both container backends can see the GPU.
ujust strix-halo-ai -- verify

# Enter a backend only when deeper troubleshooting is needed.
strix-vulkan-shell
strix-rocm-shell
```

If an option is confusing, use `--help` with the matching server wrapper to
see the full llama.cpp option reference:

```bash
llama-server-rocm --help
```

For setup, models, permissions, updates, and removal, see the main
[Strix Halo local AI guide](strix-halo-ai.md).
