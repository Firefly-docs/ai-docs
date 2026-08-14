# FAQ and Troubleshooting

This chapter covers common issues when using `llamapi-cli`, `llamapi-server`, and the APIs. Start with these checks:

```bash
llamapi platform                         # Check hardware platforms and service state
llamapi ps                               # List loaded models and automatic-loading entries
systemctl status llamapi-server          # Check the llamapi-server service state
curl -s http://127.0.0.1:9265/health     # Check API health
```

## Quick Triage

| Symptom | Troubleshooting area |
|:---:|:---:|
| `llamapi-cli` and `llamapi-server` version mismatch | [Versions and Packages](#versions-and-packages) |
| Service stopped, health failure, or no available inference platform detected | [Service Status and Platform Detection](#service-status-and-platform-detection) |
| Model listing, source, download, or variant-selection problem | [Model Discovery, Download, and Selection](#model-discovery-download-and-selection) |
| Model loading, instance, capacity, or runtime cleanup problem | [Model Loading and Instance Management](#model-loading-and-instance-management) |
| API model ID, model type, context, or multimodal input error | [HTTP API Requests](#http-api-requests) |
| Logs and environment details are needed | [Diagnostic Information](#diagnostic-information) |

## Versions and Packages

### Check `llamapi-cli` and `llamapi-server` Version Consistency

Check `llamapi-cli`:

```bash
llamapi --version
```

Check the Debian packages:

```bash
dpkg-query -W -f='${Package}: ${Version}\n' \
  firefly-llamapi-cli firefly-llamapi-server
```

`firefly-llamapi-cli` and `firefly-llamapi-server` must have the same version.

## Service Status and Platform Detection

### Service Reported as Not Running by `llamapi platform`

Check and start the service:

```bash
systemctl status llamapi-server
sudo systemctl start llamapi-server
curl -s http://127.0.0.1:9265/health
```

If the service is running but the `llamapi-cli` still reports it as stopped, an HTTP proxy may be intercepting loopback requests. The `llamapi-cli` bypasses proxies for local access; if the problem remains, check:

```bash
export no_proxy=localhost,127.0.0.1,::1
```

### Health Check Connection Failure

Check the following in order:

1. `systemctl status llamapi-server` reports a running service.
2. `/etc/llamapi-server/config.toml` uses port `9265`.
3. `journalctl -u llamapi-server -b` has no listen or configuration error.
4. No other process is using the port.

All `llamapi-cli`, configuration, and API examples in this documentation use port `9265`.

### `llamapi-server` Startup Failure

Show current-boot logs:

```bash
journalctl -u llamapi-server -b
```

If the log contains `failed to parse config file`, check:

- Table and field names.
- Closing quotes around strings.
- Double brackets for `[[models]]`.
- `instance_count` is at least `1`.
- `model_path` exists.

Restart after correcting the file:

```bash
sudo systemctl restart llamapi-server
```

### No Available Inference Platform Detected

Run:

```bash
llamapi platform
curl -s http://127.0.0.1:9265/v1/platforms
```

Common causes are:

- The required chip was not detected.
- A backend runtime library is missing or cannot load.
- The Debian package lacks a required shared library.
- The model platform does not match the device hardware.

The `llamapi-cli` only uses platform and chip combinations with `available: true` and a detected matching chip.

### Failed to Select the Inference Platform for a Model

Specify the platform and chip:

```bash
llamapi pull qwen3:4b --platform rknn3/rk1828
llamapi run qwen3:4b --platform rknn3/rk1828
```

The `llamapi-cli` rejects a combination that `/v1/platforms` does not report as available.

## Model Discovery, Download, and Selection

### Model Download Returns HTTP 401 or 404

The repository may not exist or the model name may be wrong.

1. Get model names that can be used on the current hardware:

   ```bash
   llamapi list --online
   ```

2. Copy the correct model name from the list, such as `bge-m3` instead of `bge_m3`.
3. Switch sources:

   ```bash
   llamapi pull qwen3:4b --source modelscope
   llamapi pull qwen3:4b --source huggingface
   ```

### Remote Model Lookup Timeout

The default `auto` mode queries Hugging Face and ModelScope concurrently. When all sources fail, the error reports a reason and attempt count for each source.

You can:

- Check network connectivity and DNS.
- Select a reachable source explicitly.
- Set `download.source` to `modelscope` or `huggingface` in the `llamapi-cli` configuration.
- Use `llamapi list --all` to separate repository access from `llamapi-server` platform detection.

### Multiple Platform Variants for One Model

An interactive terminal prompts for a selection. Scripts, pipes, and other non-interactive environments should specify the variant:

```bash
llamapi pull qwen3:4b --platform rkllm/rk3588
llamapi load qwen3:4b --platform rkllm/rk3588
```

`run` can select among ranked chat variants automatically, but an explicit `--platform` always takes priority.

## Model Loading and Instance Management

### `llamapi run` Reports an Embedding Model

Embedding models cannot chat. Load the model first:

```bash
llamapi load bge-m3
llamapi ps
```

Then call:

```bash
curl http://127.0.0.1:9265/v1/embeddings \
  -H "Content-Type: application/json" \
  -d '{
    "model": "bge-m3@rknn2-rk3588",
    "input": "Text to convert into a vector"
  }'
```

Use the actual runtime ID shown by `llamapi ps`.

### Service Continues Running After a Model Preload Failure

Yes. One failed `[[models]]` entry does not prevent the HTTP service from starting.

Check:

```bash
journalctl -u llamapi-server -b
curl -s http://127.0.0.1:9265/v1/models
```

Verify that:

- `model_path` points to the correct directory.
- The directory contains a valid `model.toml`.
- Model files are complete.
- The backend platform is available on the current hardware.

### API Returns `queue_full`

All instances are busy and the waiting queue is full.

Options include:

- Add instances:

  ```bash
  llamapi load <runtime-id> --instance 2
  ```

- Increase `server.request_queue_size` or the model-level queue size.
- Limit client concurrency and retry frequency.

More instances require more hardware resources, and the actual count may be lower than requested.

### Requested Model Instances Are Only Partially Loaded

The `llamapi-server` allows partial success. `llamapi-cli` and API responses report both target and actual counts:

```text
model-id partially loaded: 2/4 instances active
```

This usually indicates insufficient resources or an instance initialization failure. Check the `llamapi-server` log and use the actual count from `llamapi ps` or `/v1/models`.

### Coprocessor Communication Failure After Loading Multiple Model Instances

Loading multiple model instances on a coprocessor can cause:

- Model instance loading failure.
- Coprocessor communication failure.
- Abnormal `rknn3.service` or `llamapi-server.service` state.
- Subsequent platform queries, model loads, or inference requests to fail.

Stop loading models and sending inference requests, then execute the following recovery commands in this exact order:

```bash
sudo rknn-smi reset
sudo rknn-smi reset -t hw
systemctl restart rknn3.service
systemctl restart llamapi-server.service
```

Do not change the order: perform both chip resets first, restart `rknn3.service`, and restart `llamapi-server.service` last. If the current user cannot manage systemd services, add `sudo` to the last two commands.

Verify the recovery:

```bash
systemctl status rknn3.service
systemctl status llamapi-server.service
llamapi platform
curl -s http://127.0.0.1:9265/health
```

To avoid triggering the issue again, keep `instance_count` or `--instance` at `1` for models running on a coprocessor.

### Model Remains Loaded After an Abnormal `run` Exit

Normal completion, `Ctrl+C`, SIGTERM, and most execution errors attempt cleanup. SIGKILL, process crashes, and power loss cannot run cleanup logic.

Check and unload manually:

```bash
llamapi ps
llamapi unload <runtime-id>
```

## HTTP API Requests

### API Returns `model_not_found`

The request `model` must exactly match a loaded runtime ID.

```bash
llamapi ps
curl -s http://127.0.0.1:9265/v1/models
```

Do not substitute the local display name unless it is also the runtime ID.

### API Returns `wrong_model_type`

- `/v1/chat/completions` requires `model_kind=chat`.
- `/v1/embeddings` requires `model_kind=embedding`.

Check `model_kind` through `/v1/models`.

### API Returns `context_length_exceeded`

The request content and expected output exceed the model context limit.

- Reduce conversation history or input content.
- Reduce `max_tokens` or `max_completion_tokens`.
- Check the model's `max_context_len` setting.

### Remote Image URL Unsupported

`image_url` currently supports:

- A local file path on the `llamapi-server`.
- A base64 data URL for `png`, `jpeg`, `jpg`, or `webp`.

It does not support `http://` or `https://` images. Download the image first or encode it as a data URL.

## Diagnostic Information

### Collecting Diagnostic Information

Collect:

```bash
llamapi --version
llamapi platform
llamapi list
llamapi ps
systemctl status llamapi-server
journalctl -u llamapi-server -b
curl -s http://127.0.0.1:9265/v1/platforms
curl -s http://127.0.0.1:9265/v1/models
```

Also record the device, chip, command, complete error output, and relevant `llamapi-server` configuration. Review logs for sensitive paths or business data before sharing them.
