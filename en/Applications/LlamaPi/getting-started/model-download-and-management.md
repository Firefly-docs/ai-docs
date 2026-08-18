# Download and Manage Models

This guide explains how to find models available on the current hardware and how to download, inspect, and remove local models.

## Model Names and Inference Platforms

### Model Names and Types

LlamaPi accepts model names with or without a parameter size:

| Format | Example |
|:---:|:---:|
| `<model>:<size>` | `qwen3:4b` |
| `<model>` | `bge-m3` |

Models currently have two types:

| Type | Purpose |
|:---:|:---:|
| `chat` | Chat, text generation, and tool calls |
| `embedding` | Convert text into vectors |

### Currently Supported Inference Platforms

| Inference platform | Model type | Currently supported hardware |
|:---:|:---:|:---:|
| `rknn3` | `chat` | RK1820 and RK1828 |
| `rkllm` | `chat` | RK3588 |
| `rknn2` | `embedding` | RK3588 |

One model may have variants for different inference platforms and hardware. LlamaPi normally selects a variant for the current hardware. To select one manually, use `--platform <platform>/<chip>`.

LlamaPi detects the current hardware and available inference platforms. Future releases will add support for more hardware and inference platforms.

## Find and Download Models

### Find Models Available on the Current Hardware

```bash
llamapi list --online
```

This command lists all downloadable models available on the current hardware.

### Download a Model

Run the following command to automatically select the recommended platform and download the corresponding model:

```bash
llamapi pull qwen3:4b
```

You can add the `--platform` option to download the model for a specified inference platform:

```bash
llamapi pull qwen3:4b --platform rkllm/rk3588
```

After the download, LlamaPi validates the model files and displays the model size.

## Manage Local Models

### List Local Models

```bash
llamapi list
```

Local models are stored in this directory by default:

```text
/var/lib/llamapi/models
```

### Remove a Local Model

```bash
llamapi rm qwen3:4b
```

Remove a model variant for a specific inference platform:

```bash
llamapi rm qwen3:4b --platform rkllm/rk3588
```

If the model is running, LlamaPi asks whether to unload it first. After removing the local files, download the model again before using it.

Continue with [Run and Deploy Models](./model-load-and-run.md). See the [Terminal Command Guide](../advanced-guides/cli-command-guide.md) for complete command options.
