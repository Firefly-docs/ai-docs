# Load and Run Models

This guide explains how to run chat models, keep models loaded for ongoing service, inspect loaded models, and unload them.

## Run a Chat Model

Open an interactive chat:

```bash
llamapi run qwen3:4b
```

Send a single prompt:

```bash
llamapi run qwen3:4b "Explain what an NPU is."
```

Set a system prompt:

```bash
llamapi run qwen3:4b --system "You are an embedded Linux engineer."
```

As needed, `run` downloads and loads the model. It unloads a model loaded by the current command when the command exits. Use it for temporary chats and single prompts.

## Use Interactive Mode

The following commands are available in interactive mode:

| Command | Description |
|:---:|:---:|
| `/help` | Show available commands |
| `/clear` | Clear conversation history |
| `/system <prompt>` | Set or clear the system prompt |
| `/set <key> <value>` | Change a generation parameter |
| `/show info` | Show current model information |
| `/show params` | Show current generation parameters |
| `/exit` | Exit the chat |

Enter `"""` to start or finish multiline input. Press `Ctrl+C` during generation to interrupt the current response.

## Load a Model

To keep a model available across multiple requests, run:

```bash
llamapi load qwen3:4b
```

Load multiple instances:

```bash
llamapi load qwen3:4b --instance 2
```

Set a runtime ID:

```bash
llamapi load qwen3:4b --id assistant
```

Third-party applications and API requests should use the actual runtime ID shown by `llamapi ps`.

## List Loaded Models

```bash
llamapi ps
```

This command displays each model's runtime ID, type, inference platform, state, instance count, and automatic-loading state.

## Resize Instances

Resize a loaded model to three instances:

```bash
llamapi load assistant --instance 3
```

More instances can increase parallel request capacity but consume more hardware resources. Select a count appropriate for the device.

## Unload a Model

```bash
llamapi unload assistant
```

Unloading a model does not remove its local files.

## Use an Embedding Model

Embedding models cannot be used for chat through `llamapi run`. Download and load the model first:

```bash
llamapi pull bge-m3
llamapi load bge-m3
llamapi ps
```

Then call `/v1/embeddings`. See the [API Reference](../advanced-guides/api-reference.md#embeddings) for request details.

To load a model automatically when the LlamaPi service starts, continue with [Persistent Deployment](./model-persistence.md).
