# Service Configuration and Operations

`llamapi-server` is the LlamaPi server component. It detects inference platforms, loads models, manages model instances, and provides OpenAI-compatible HTTP APIs.

This chapter assumes that `firefly-llamapi-server` has already been installed as described in [Quick Start](../getting-started/quickstart.md). It focuses on service management, configuration, logging, model preloading, and troubleshooting.

## Service Information

| Item | Value |
|:---:|:---:|
| Debian package | `firefly-llamapi-server` |
| systemd unit | `llamapi-server.service` |
| Binary | `/usr/bin/llamapi-server` |
| Default configuration | `/etc/llamapi-server/config.toml` |
| Default listen address | `0.0.0.0:9265` |
| Health check | `GET /health` |

systemd starts the service in the following form:

```text
ExecStart=/usr/bin/llamapi-server --config /etc/llamapi-server/config.toml
```

## Manage the Service

| Operation | Command |
|:---:|:---:|
| Show status | `systemctl status llamapi-server` |
| Start | `sudo systemctl start llamapi-server` |
| Stop | `sudo systemctl stop llamapi-server` |
| Restart | `sudo systemctl restart llamapi-server` |
| Enable at boot | `sudo systemctl enable llamapi-server` |
| Disable at boot | `sudo systemctl disable llamapi-server` |
| Reload systemd units | `sudo systemctl daemon-reload` |

Restart the service after changing its configuration:

```bash
sudo systemctl restart llamapi-server
```

## View Logs

Follow logs in real time:

```bash
journalctl -u llamapi-server -f
```

Show logs from the current system boot:

```bash
journalctl -u llamapi-server -b
```

Show the latest 100 lines:

```bash
journalctl -u llamapi-server -n 100
```

Check the service log first when diagnosing startup, configuration parsing, platform loading, or model preloading failures.

## Command-Line Options

`llamapi-server` supports:

| Option | Description |
|:---:|:---:|
| `--host <HOST>` | Override the listen address |
| `-p, --port <PORT>` | Override the listen port |
| `-c, --config <CONFIG>` | Select a TOML configuration file |
| `--log-level <LEVEL>` | Set `trace`, `debug`, `info`, `warn`, or `error` |
| `-h, --help` | Show help |
| `-V, --version` | Show the version, commit hash, and build time |

Configuration priority, from highest to lowest, is:

1. Command-line options.
2. Configuration file.
3. Built-in defaults.

The systemd service uses the configuration file. Except for temporary debugging, keep persistent settings in `/etc/llamapi-server/config.toml`.

## Configuration File

Default path:

```text
/etc/llamapi-server/config.toml
```

Basic configuration:

```toml
[server]
host = "0.0.0.0"
port = 9265
log_level = "info"
```

Complete example with global generation defaults and model preloading:

```toml
[server]
host = "0.0.0.0"
port = 9265
log_level = "info"
request_queue_size = 48

[defaults]
temperature = 1.0
top_p = 0.9
top_k = 1
repeat_penalty = 1.2
frequency_penalty = 0.0
presence_penalty = 0.0
max_tokens = 1024
max_context_len = 4096
stop = []
enable_thinking = false

[[models]]
model_id = "qwen3:4b@rkllm-rk3588"
model_path = "/var/lib/llamapi/models/rkllm/rk3588/qwen3-4b"
instance_count = 1
request_queue_size = 48

[models.default_params]
temperature = 0.7
top_p = 0.9
max_tokens = 1024
max_context_len = 4096
stop = []
enable_thinking = false
```

### `[server]` Settings

| Field | Default | Description |
|:---:|:---:|:---:|
| `host` | `0.0.0.0` | HTTP listen address |
| `port` | `9265` | HTTP listen port |
| `log_level` | `info` | Log level |
| `request_queue_size` | `48` | Default waiting capacity for each model |

Approximate total request capacity for a model is:

```text
actual instance count + request_queue_size
```

When all instances are busy and the waiting queue is full, the `llamapi-server` returns HTTP `429` with error code `queue_full`.

### `[defaults]` Settings

`[defaults]` supplies global generation defaults. A model's `model.toml` and `[[models]].default_params` can override these values.

| Field | Description |
|:---:|:---:|
| `temperature` | Sampling temperature |
| `top_p` | Nucleus sampling value |
| `top_k` | Top-k sampling value |
| `repeat_penalty` | Repetition penalty |
| `frequency_penalty` | Frequency penalty |
| `presence_penalty` | Presence penalty |
| `max_tokens` | Default maximum generated tokens |
| `max_context_len` | Runtime context limit |
| `stop` | Stop-sequence array |
| `enable_thinking` | Enable the model's thinking mode |

Matching fields in an inference request override model defaults. See the [API Reference](./api-reference.md#request-fields).

### `[[models]]` Settings

Each `[[models]]` entry defines a model group that is preloaded when the `llamapi-server` starts.

| Field | Required | Description |
|:---:|:---:|:---:|
| `model_id` | Yes | Runtime model ID exposed to clients |
| `model_path` | Yes | Model directory path |
| `instance_count` | No | Target instance count; default `1`, minimum `1` |
| `request_queue_size` | No | Per-model queue capacity; uses the `llamapi-server` default when omitted |
| `default_params` | No | Per-model generation defaults |

The `llamapi-cli` can manage these entries:

```bash
llamapi enable qwen3:4b --instance 2
llamapi disable qwen3:4b
```

Prefer the `llamapi-cli` when possible so that model paths and runtime IDs are generated consistently.

**Coprocessor limitation**: Set `instance_count` to `1` for a coprocessor model. Multiple instances may cause model loading failure, coprocessor communication failure, and abnormal `rknn3.service` and `llamapi-server.service` state. See [Coprocessor Communication Failure After Loading Multiple Model Instances](../llamapi/faq.md#coprocessor-communication-failure-after-loading-multiple-model-instances) for recovery.

## Model Preloading

At startup, the `llamapi-server` reads and loads `[[models]]` entries in order.

- A single preload failure is written to the log.
- One failed model does not prevent the HTTP service from starting.
- The actual number of created instances may be lower than the requested number.
- `instance_count` in `/v1/models` is the number of instances currently able to serve requests.

Check preloading results:

```bash
curl -s http://127.0.0.1:9265/v1/models
```

Important response fields:

| Field | Description |
|:---:|:---:|
| `id` | Model ID used by clients |
| `platform` | Inference platform |
| `instance_count` | Active instance count |
| `model_path` | Source model directory |
| `model_kind` | `chat` or `embedding` |

## Runtime Model Management

Models can also be managed while the `llamapi-server` is running:

| Operation | API | `llamapi-cli` command |
|:---:|:---:|:---:|
| Load | `POST /v1/models/load` | `llamapi load` |
| Resize | `POST /v1/models/resize` | `llamapi load <id> --instance <N>` |
| Unload | `POST /v1/models/unload` | `llamapi unload` |

The `llamapi-cli` resolves model names, paths, and runtime IDs. Direct API calls must provide the full model path on the `llamapi-server` filesystem.

## Health Checks

The health endpoint does not depend on any loaded model:

```bash
curl -s http://127.0.0.1:9265/health
```

Response:

```text
ok
```

`ok` only means that the HTTP service is running. To check model readiness, also use:

```bash
curl -s http://127.0.0.1:9265/v1/models
curl -s http://127.0.0.1:9265/v1/platforms
```

## Network Access and Security

The current `llamapi-server`:

- Does not require API authentication.
- Allows cross-origin requests.
- Listens on `0.0.0.0:9265` by default.

Before exposing it to an untrusted network, restrict access with a firewall, reverse proxy, or controlled network boundary.

## Troubleshooting

| Symptom | Check | Resolution |
|:---:|:---:|:---:|
| `/health` cannot connect | Service status and port | Run `systemctl status llamapi-server`; confirm port `9265` |
| Service startup fails | TOML parsing | Check `journalctl -u llamapi-server -b` for `failed to parse config file` |
| Platform is unavailable | Backend runtime and hardware | Verify required runtime libraries and package contents |
| Model preload fails | `model_path`, `model.toml`, model files | Check the log for `Failed to preload model` |
| Coprocessor communication fails after loading multiple instances | Whether multiple instances were configured on a coprocessor | Stop loading models and follow the [recovery procedure](../llamapi/faq.md#coprocessor-communication-failure-after-loading-multiple-model-instances) to reset the chip and restart services in order |
| `model_not_found` | Loaded state and runtime ID | Call `/v1/models` or run `llamapi ps` |
| `queue_full` | Busy instances and full queue | Add instances or queue capacity, or reduce concurrency |
| `wrong_model_type` | API and model type | Use `chat` models for chat and `embedding` models for Embeddings |
| `invalid_instance_count` | Instance count is `0` | Set it to `1` or greater |
| Listen failure | Port conflict | Stop the process using `9265` or coordinate service ports |

See [FAQ and Troubleshooting](../llamapi/faq.md) for issues that span the `llamapi-cli`, `llamapi-server`, and APIs.
