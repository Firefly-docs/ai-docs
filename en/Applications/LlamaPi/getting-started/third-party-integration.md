# Connect Third-Party Applications

LlamaPi can provide local large-model capabilities to third-party applications through OpenAI-compatible APIs.

Applications that support a custom OpenAI Base URL can usually connect by supplying the LlamaPi service URL and model ID. If the application requires an API key, enter any non-empty value.

This guide uses a chat model to explain the general integration flow. See the [API Reference](../advanced-guides/api-reference.md) for complete fields and supported API capabilities.

## Prerequisites

Before connecting a third-party application, follow the steps below to verify that LlamaPi is running properly, that the network is reachable, and that the API endpoint is available.

### Check the Service and Inference Platform

```bash
llamapi platform
```

Example of a successful check:

```text
SoC:                    RK3588
Coprocessor:            null
Supported platforms:    rkllm, rknn2
Unsupported platforms:  rknn3
Service:                running  (http://127.0.0.1:9265/v1)
```

The LlamaPi service and inference platforms are operating normally when the output identifies the `SoC`, lists at least one entry under `Supported platforms`, and shows `Service: running`.

`Coprocessor: null` is normal on devices without a coprocessor. Entries under `Unsupported platforms` do not prevent the supported platforms from being used.

### Check That the Required Model Is Loaded

```bash
llamapi ps
```

Example of a successful check:

```text
ID                         MODEL       TYPE   PLATFORM       STATUS       INSTANCES    ENABLE
qwen3:4b@rkllm-rk3588      qwen3:4b    chat   rkllm/rk3588   ● active     1            no
```

Find the model you plan to connect and verify that:

- The `MODEL` column matches the model you want to use. If multiple rows have the same model name, use the `PLATFORM` column to distinguish the inference-platform variants.
- A model with `TYPE` set to `chat` can receive chat requests, while a model with `TYPE` set to `embedding` can receive text embedding requests.
- `STATUS` set to `● active` indicates that the model is loaded and can receive requests of the corresponding type.

After confirming the model, use the complete value from the `ID` column as the model name in the third-party application. Do not enter only the display name from the `MODEL` column.

If the required model is missing or its status is `○ inactive`, [load the model](./model-load-and-run.md) first:

```bash
llamapi load qwen3:4b
```

### Check Service Reachability

#### Access LlamaPi Locally

If the third-party application and LlamaPi run on the same device, use the following service URL:

```text
http://127.0.0.1:9265/v1
```

On the device's Linux terminal, execute:

```bash
curl -s http://127.0.0.1:9265/health
```

On Windows, execute the following in PowerShell or Command Prompt:

```powershell
curl.exe -s http://127.0.0.1:9265/health
```

#### Access LlamaPi over a LAN

If the third-party application runs on another device on the same LAN, first check the IP address of the device running LlamaPi:

```bash
hostname -I
```

Replace `<device-ip>` in the following address with the actual IP address you queried:

```text
http://<device-ip>:9265/v1
```

From the device running the third-party application, test connectivity to the service. If that device runs Linux, execute:

```bash
curl -s http://<device-ip>:9265/health
```

If that device runs Windows, execute the following in PowerShell or Command Prompt:

```powershell
curl.exe -s http://<device-ip>:9265/health
```

A response of `ok` means that the application device can reach the LlamaPi service. If the connection times out or is refused, check the device IP, firewall, and port `9265`.

### Verify the Inference Endpoints

From a terminal that can access the LlamaPi service, verify the chat or embeddings endpoint depending on your integration needs.

Replace `<device-ip>` in the following commands with `127.0.0.1` for local access, or with the actual IP address of the device running LlamaPi for LAN access.

Chat endpoint (for models with `TYPE` set to `chat`):

```bash
curl http://<device-ip>:9265/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "qwen3:4b@rkllm-rk3588",
    "messages": [
      {"role": "user", "content": "Hello"}
    ],
    "stream": false
  }'
```

Embeddings endpoint (for models with `TYPE` set to `embedding`):

```bash
curl http://<device-ip>:9265/v1/embeddings \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "bge-m3@rknn2-rk3588",
    "input": "text to be converted into a vector"
  }'
```

Replace `model` with the actual model ID displayed by `llamapi ps`.

## Configure the Third-Party Application

Setting names vary between applications, but the general process is:

1. Select OpenAI or an OpenAI-compatible provider in the application.
2. Find the custom Base URL, API endpoint, or service URL setting.
3. Enter the Base URL and model ID confirmed in the preceding steps. If the application requires an API key, enter any non-empty value.
4. Save the configuration and send a test message.

Example settings:

| Setting | Value |
|:---:|:---:|
| OpenAI Base URL | `http://<device-ip>:9265/v1` |
| Model | Model ID shown by `llamapi ps` (for example, `qwen3:4b@rkllm-rk3588`) |
| API key | Any non-empty value if the application requires one |

If the application discovers models automatically, confirm that it requests `/v1/models`. Applications that do not allow a custom Base URL, or that accept only a specific cloud service URL, cannot connect through this general method.

> ⚠️ <strong style="color: #dc2626;">Security reminder</strong>: LlamaPi currently does not provide API authentication.
>
> An API key entered in a third-party application only satisfies the client's required-field validation; LlamaPi does not verify it, and it does not restrict access.
>
> Use the service only on the local device or a trusted LAN. Do not expose port `9265` directly to the public internet.
>
> For access across untrusted networks, add a firewall, reverse proxy, and authentication. See [Service Configuration and Operations](../advanced-guides/server-operations.md#network-access-and-security).

## Common Connection Issues

| Symptom | What to check |
|:---:|:---:|
| Cannot connect to the service | Verify the device IP, port, LlamaPi service state, and network connectivity |
| Model cannot be found | Run `llamapi ps`, confirm the model is loaded, and use the complete model ID |
| API key is reported as empty | Enter any non-empty value in the third-party application |
| Chat returns a model-type error | Use a `chat` model rather than an `embedding` model |
| Device is reachable but API requests fail | Check whether the device firewall allows access to port `9265` |

See [FAQ and Troubleshooting](../llamapi/faq.md) for additional help.
