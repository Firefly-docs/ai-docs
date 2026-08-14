# Download and Manage Models

This guide explains how to find models available on the current hardware and how to download, inspect, and remove local models.

## Model Names and Types

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

One model may have variants for different inference platforms and hardware. LlamaPi normally selects a variant for the current hardware. To select one manually, use `--platform <platform>/<chip>`.

## Currently Supported Inference Platforms

| Inference platform | Model type | Currently supported hardware |
|:---:|:---:|:---:|
| `rknn3` | `chat` | RK1820 and RK1828 coprocessors, or the RK3572 SoC |
| `rkllm` | `chat` | RK3576, RK3588, RK3562, RV1126B, and other SoCs |
| `rknn2` | `embedding` | NPU platforms supported by RKNN2 |

LlamaPi detects the current hardware and available inference platforms. Future releases will add support for more hardware and inference platforms.

## Find Models Available on the Current Hardware

```bash
llamapi list --online
```

This command shows only remote models available on the current hardware. Model data comes from Hugging Face or ModelScope by default.

To list all remote models without filtering for the current hardware:

```bash
llamapi list --all
```

## Download a Model

```bash
llamapi pull qwen3:4b
```

Select an inference platform and hardware:

```bash
llamapi pull qwen3:4b --platform rkllm/rk3588
```

Select a download source:

```bash
llamapi pull qwen3:4b --source modelscope
```

After the download, LlamaPi validates the model files and displays the model size. Downloading a model does not load it.

## List Local Models

```bash
llamapi list
```

Local models are stored in this directory by default:

```text
/var/lib/llamapi/models
```

## Remove a Local Model

```bash
llamapi rm qwen3:4b
```

Remove a model variant for a specific inference platform:

```bash
llamapi rm qwen3:4b --platform rkllm/rk3588
```

If the model is running, LlamaPi asks whether to unload it first. After removing the local files, download the model again before using it.

Continue with [Load and Run Models](./model-load-and-run.md). See the [Terminal Command Guide](../advanced-guides/cli-command-guide.md) for complete command options.
