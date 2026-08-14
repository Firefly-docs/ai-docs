# Terminal Command Guide

`llamapi-cli` is the LlamaPi command-line component. Its command entry point is `llamapi`, which provides hardware inspection, model discovery and download, chat execution, model instance management, and automatic-load configuration.

For task-oriented workflows, read [Download and Manage Models](../getting-started/model-download-and-management.md), [Load and Run Models](../getting-started/model-load-and-run.md), and [Persistent Deployment](../getting-started/model-persistence.md) first. This chapter documents the syntax, options, prerequisites, behavior, and output of every command.

## Command Format

```text
llamapi [OPTIONS] <COMMAND>
```

Available commands:

| Command | Purpose |
|:---:|:---:|
| `run` | Run a chat model interactively or send a single prompt |
| `list` | List local or remote models |
| `pull` | Download a model |
| `rm` | Remove a local model |
| `load` | Load a model or resize its instance count |
| `unload` | Unload a model group |
| `ps` | Show runtime and automatic-load status |
| `platform` | Show hardware, platform, and service status |
| `enable` | Configure a model to load when the `llamapi-server` starts |
| `disable` | Remove automatic-load configuration |

## Global Options

| Option | Description |
|:---:|:---:|
| `--config <PATH>` | Use a `llamapi-cli` configuration file; default: `~/.config/llamapi-cli/config.toml` |
| `-h, --help` | Show help |
| `-V, --version` | Show the version |

`--config` is a top-level option and must appear before the subcommand:

```bash
llamapi --config /path/to/config.toml load qwen3:4b
```

Check the version:

```bash
llamapi --version
```

Current output is similar to:

```text
✦ Firefly AI · LlamaPi v0.2.0
```

`firefly-llamapi-cli` and `firefly-llamapi-server` always have the same version.

## `llamapi-cli` Configuration

Default path:

```text
~/.config/llamapi-cli/config.toml
```

When the file is absent, the `llamapi-cli` uses defaults. A complete example is:

```toml
[model_store]
path = "/var/lib/llamapi/models"

[download]
source = "auto"

[server]
config_path = "/etc/llamapi-server/config.toml"
```

| Field | Default | Description |
|:---:|:---:|:---:|
| `model_store.path` | `/var/lib/llamapi/models` | Local model storage root |
| `download.source` | `auto` | `auto`, `modelscope`, or `huggingface` |
| `server.config_path` | `/etc/llamapi-server/config.toml` | `llamapi-server` configuration path |

The `llamapi-cli` reads `host` and `port` from the `llamapi-server` configuration. A bind address such as `0.0.0.0` is converted to `127.0.0.1` for local access. All examples use port `9265`.

## Model Argument Conventions

### Model Names

Supported forms include:

```text
qwen3:4b
bge-m3
```

Model variants are displayed as:

```text
model(platform/chip)
```

For example:

```text
qwen3:4b(rkllm/rk3588)
```

### Platform Option

Use the following option when a variant must be selected explicitly:

```text
-p, --platform <platform>/<chip>
```

Platform and chip names are case-sensitive. Use values reported by `llamapi platform` and `llamapi list --online`.

### Runtime IDs

The default runtime ID is:

```text
name@platform-chip
```

A custom ID:

- May contain Chinese characters.
- Cannot contain whitespace.
- Cannot contain `/`.
- Cannot have a display width greater than 20.
- Cannot conflict with a loaded model group or automatic-load entry.

## Environment and Status

### `platform`

Shows hardware, inference platforms, and `llamapi-server` status.

```bash
llamapi platform
```

The command has no options.

Example with a running service:

```text
SoC:                    RK3588
Coprocessor:            null
Supported platforms:    rkllm, rknn2
Unsupported platforms:  rknn3
Service:                running  (http://127.0.0.1:9265/v1)
```

Example when the service is stopped:

```text
SoC:                    unknown
Coprocessor:            unknown
Supported platforms:    unknown (service not running)
Unsupported platforms:  unknown (service not running)
Service:                not running  (http://127.0.0.1:9265/v1)

Run 'llamapi run <model>' to start the service.
```

Notes:

- SoC, coprocessor, and platform data come from `GET /v1/platforms`.
- Multiple identical chips are displayed as `model ×count`.
- An undetected coprocessor is displayed as `null`.
- A running service with no available platform reports `none`.
- A stopped service cannot provide hardware data, so related values are `unknown`.

### `list`

Lists models. Without options, it lists downloaded local models.

```text
llamapi list [OPTIONS]
```

| Option | Description |
|:---:|:---:|
| `-o, --online` | List remote models compatible with current hardware |
| `-a, --all` | List all remote models without hardware filtering |

Examples:

```bash
llamapi list
llamapi list --online
llamapi list --all
```

Output columns:

| Column | Description |
|:---:|:---:|
| `MODEL` | Display model name |
| `TYPE` | `chat`, `embedding`, or `-` when unknown |
| `PLATFORM` | `<platform>/<chip>` |
| `SIZE` | Model size, or `-` when no reliable value is available |
| `LOCAL` | `● installed` or `○ not installed` |

`list --online` depends on `/v1/platforms` and does not start the `llamapi-server`. It only shows variants with `available: true` and a matching detected chip.

`list --all` does not filter by hardware and does not depend on the platform API.

Remote lookup behavior:

- `auto` queries Hugging Face and ModelScope concurrently and uses the first valid result.
- Success from one source cancels the unfinished request to the other source.
- Each source is attempted up to three times, with 500 ms and 1000 ms delays before retries.
- Connection timeout is 3 seconds, each request timeout is 6 seconds, and the total lookup limit is 20 seconds.
- Connection, send, read, parse, HTTP `429`, and HTTP `5xx` failures are retried.
- Other `4xx` responses and empty repository lists are not retried.

Sorting is stable and natural: first by `MODEL`, then by `PLATFORM`. Models without a size suffix precede sized variants, and sizes are ordered numerically.

### `ps`

Shows loaded models and automatic-load configuration.

```bash
llamapi ps
```

The command has no options.

Example:

```text
ID       MODEL       TYPE        PLATFORM          STATUS        INSTANCES    ENABLE
abc      qwen3:4b    chat        rkllm/rk3588      ● active      2            yes(1)
demo     qwen3:4b    chat        rkllm/rk3588      ○ inactive    -            yes(1)
```

| Column | Description |
|:---:|:---:|
| `ID` | Runtime model ID |
| `MODEL` | Display model name |
| `TYPE` | `chat` or `embedding` |
| `PLATFORM` | Platform and chip |
| `STATUS` | `● active` or `○ inactive` |
| `INSTANCES` | Active instance count, or `-` when inactive |
| `ENABLE` | `yes(N)` means N automatic-load instances |

A model that is only downloaded, with no loaded runtime ID and no automatic-load entry, is not shown. When there are no matching models, only the table header is printed.

## Local Model Management

### `pull`

Downloads a model into the local model store.

```text
llamapi pull <MODEL> [OPTIONS]
```

| Argument or option | Description |
|:---:|:---:|
| `MODEL` | Model name, such as `qwen3:4b` |
| `-p, --platform <PLATFORM>` | Platform and chip, such as `rkllm/rk3588` |
| `--source <SOURCE>` | `auto`, `modelscope`, or `huggingface` |

Examples:

```bash
llamapi pull qwen3:4b
llamapi pull qwen3:4b --platform rknn3/rk1828
llamapi pull qwen3:4b --source modelscope
```

Execution flow:

1. Query `/v1/platforms` for available platforms and chips.
2. Select a remote repository from the model name and platform.
3. Prompt in an interactive terminal when multiple compatible candidates exist; require `--platform` otherwise.
4. Ask whether to redownload an existing model.
5. Download files with progress output.
6. Validate `model.toml`.
7. Display total model size.

`pull` does not start the `llamapi-server` and does not load the model after downloading.

Successful output is similar to:

```text
✦ Firefly AI · LlamaPi
✔ Download complete: qwen3:4b(rkllm/rk3588)
  Platform  rkllm/rk3588
  Size      3.8 GB
```

### `rm`

Removes local model files.

```text
llamapi rm <MODEL> [OPTIONS]
```

| Argument or option | Description |
|:---:|:---:|
| `MODEL` | Model name |
| `-p, --platform <PLATFORM>` | Select the platform variant to remove |

Examples:

```bash
llamapi rm qwen3:4b
llamapi rm qwen3:4b --platform rkllm/rk3588
```

If matching runtime IDs are active, the `llamapi-cli` asks whether to unload them. After confirmation, it removes the model directory and displays the freed disk space.

Multiple local variants prompt interactively; non-interactive use requires an explicit platform.

## Model Execution

### `run`

Starts the `llamapi-server` as needed, downloads and loads a chat model, and then opens an interactive session or sends a single prompt.

```text
llamapi run <MODEL_OR_ID> [PROMPT] [OPTIONS]
```

| Argument or option | Description |
|:---:|:---:|
| `MODEL_OR_ID` | Model name or loaded runtime ID |
| `PROMPT` | Optional single prompt |
| `-p, --platform <PLATFORM>` | Select a platform and chip |
| `--system <TEXT>` | Set the system prompt |
| `--temperature <FLOAT>` | Override temperature |
| `--top-p <FLOAT>` | Override top-p |
| `--max-tokens <INT>` | Override maximum generated tokens |
| `--stats` | Show token counts, first-token latency, and generation speed |

Examples:

```bash
llamapi run qwen3:4b
llamapi run qwen3:4b "Hello"
llamapi run qwen3:4b --system "You are a Python assistant."
llamapi run qwen3:4b --platform rknn3/rk1828
llamapi run abc
llamapi run qwen3:4b "Write a poem" --temperature 0.9 --max-tokens 512
llamapi run qwen3:4b "Hello" --stats
```

Key behavior:

1. Starts the `llamapi-server` if it is not running.
2. Uses an exact loaded runtime ID directly.
3. Strictly uses an explicit platform without automatic performance selection.
4. Prefers compatible local chat variants when no platform is specified.
5. Queries remote repositories only when no local variant is available.
6. Rejects chat when only an Embedding variant exists and suggests `load`.
7. Downloads a missing model automatically.
8. Uses the default runtime ID for model-name execution.
9. Reuses an existing default ID that points to the same model directory without resizing it.
10. Reports a conflict if the default ID points to another directory.
11. Unloads only a model newly loaded by the current command.
12. Does not stop a `llamapi-server` that it started automatically.

With `--stats`, the following statistics are displayed after the response:

```text
--- stats ---
prompt_tokens:     13
completion_tokens: 14
first_token:       380 ms
speed:             18.7 token/s
```

#### Concurrency behavior

Multiple `run` processes share one runtime model group, but version `0.2.0` has no session lease or reference count. The process that loaded the model may unload it while another process is still using it.

For stable concurrent serving:

1. Preload the model with `llamapi load`.
2. Have clients call the HTTP API directly.
3. Configure instance count and request queue for the available hardware.

#### Interactive Mode Commands

`llamapi run <MODEL>` enters interactive mode.

| Command | Description |
|:---:|:---:|
| `/help` | Show REPL help |
| `/clear` | Clear conversation history |
| `/system <prompt>` | Set the system prompt; clear it with no argument |
| `/set <key> <value>` | Set a generation parameter |
| `/show info` | Show model information |
| `/show params` | Show current generation parameters |
| `/show system` | Show the system prompt |
| `/exit` | Exit the session |

`/set` supports:

| Parameter | Value |
|:---:|:---:|
| `temperature` | Floating-point number |
| `top_p` | Floating-point number |
| `max_tokens` | Integer |
| `enable_thinking` | `true` or `false` |

Enter `"""` to start multiline input and enter it again to finish. Press `Ctrl+C` during generation to interrupt the current response. Press `Ctrl+C` at the input prompt or `Ctrl+D` to exit.

## Model Instance Management

### `load`

Loads a local model or resizes an existing runtime ID.

```text
llamapi load <MODEL_OR_ID> [OPTIONS]
```

| Argument or option | Default | Description |
|:---:|:---:|:---:|
| `MODEL_OR_ID` | — | Local model name or active runtime ID |
| `-p, --platform <PLATFORM>` | Automatic | Select a local variant; validate an ID branch platform |
| `--instance <N>` | `1` | Target instance count, at least 1 |
| `--id <ID>` | Default runtime ID | Custom ID for a newly loaded model |

Examples:

```bash
llamapi load qwen3:4b
llamapi load qwen3:4b --instance 2
llamapi load qwen3:4b --platform rkllm/rk3588 --id abc --instance 2
llamapi load abc --instance 3
```

Behavior:

- An exact active ID is resized to `--instance`.
- A local model name loads the requested number of instances.
- An existing default ID for the same model is resized.
- Matching instance counts result in an already-loaded message and no operation.
- Partial instance creation can still succeed and reports both requested and actual counts.
- `--id` cannot be used when the positional argument already matches an existing ID.
- Multiple local variants prompt interactively; non-interactive use requires `--platform`.

The `llamapi-server` must be running. A model-name load requires the model to be downloaded locally.

**Coprocessor limitation**: Using `--instance` to load multiple model instances on a coprocessor may cause loading failure, chip communication failure, and abnormal service state. Keep `--instance 1`. See [Coprocessor Communication Failure After Loading Multiple Model Instances](../llamapi/faq.md#coprocessor-communication-failure-after-loading-multiple-model-instances) for recovery.

### `unload`

Unloads a model group and all of its instances.

```text
llamapi unload <MODEL_OR_ID> [OPTIONS]
```

| Argument or option | Description |
|:---:|:---:|
| `MODEL_OR_ID` | A model name or ID shown by `llamapi ps` |
| `-p, --platform <PLATFORM>` | Require an exact platform and chip match |

Examples:

```bash
llamapi unload qwen3:4b
llamapi unload abc
llamapi unload qwen3:4b --platform rkllm/rk3588
```

Matching rules:

1. Match an exact ID first.
2. If no ID matches, match by model name.
3. Unload directly when one model group matches.
4. Prompt interactively when multiple groups match.
5. Require an explicit ID in non-interactive use.

IDs, model names, and platforms are case-sensitive. The `llamapi-server` must be running.

## Automatic Loading

### `enable`

Adds a model to the `llamapi-server` configuration so it loads whenever the service starts.

```text
llamapi enable <MODEL> [OPTIONS]
```

| Argument or option | Default | Description |
|:---:|:---:|:---:|
| `MODEL` | — | Local model name |
| `-p, --platform <PLATFORM>` | Automatic | Select a platform and chip |
| `--instance <N>` | `1` | Automatic-load instance count |
| `--id <ID>` | Default runtime ID | Custom runtime ID |
| `--now` | — | Load immediately after updating configuration |

Examples:

```bash
llamapi enable qwen3:4b
llamapi enable qwen3:4b --instance 2 --now
llamapi enable qwen3:4b --platform rkllm/rk3588 --id abc --instance 2
```

If an ID already points to the same model directory, the command updates its instance count and defaults. If the ID points to another directory, the command reports a conflict.

Multiple local variants prompt interactively; non-interactive use requires an explicit platform.

### `disable`

Removes a model automatic-load entry from the `llamapi-server` configuration.

```text
llamapi disable <MODEL_OR_ID> [OPTIONS]
```

| Argument or option | Description |
|:---:|:---:|
| `MODEL_OR_ID` | Model name or configured ID |
| `-p, --platform <PLATFORM>` | Select a platform and chip |
| `--now` | Unload the model after removing the entry |

Examples:

```bash
llamapi disable qwen3:4b
llamapi disable qwen3:4b --now
llamapi disable abc
```

An exact configured ID removes that ID. Otherwise, the `llamapi-cli` derives the default ID from the model name and platform. A custom ID must be specified explicitly and is not removed indirectly by `disable qwen3:4b`.
