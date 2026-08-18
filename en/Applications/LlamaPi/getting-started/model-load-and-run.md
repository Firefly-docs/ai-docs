# Run and Deploy Models

This guide explains how to use `run` to run models and `load` to deploy them.

[Downloading a model](./model-download-and-management.md) only stores its files locally. Before inference can begin, LlamaPi must load the model onto the appropriate inference platform and create a runtime model that can receive requests. A loaded model consumes memory resources on the device.

Both `run` and `load` load a model, but they serve different purposes and have different lifecycles:

| Method | Behavior | Use case |
|:---:|:---:|:---:|
| `llamapi run` | Starts the service, downloads, and loads the model as needed; unloads a model newly loaded by the command when it exits | Interactive chats and single prompts |
| `llamapi load` | Loads a downloaded model; keeps it deployed after the command exits until it is unloaded or the service stops | Ongoing API service and third-party application integration |

If `run` reuses a model already deployed with `load`, exiting `run` does not unload that model.

## Run a Model (`run`)

The `run` command automatically finds, downloads when necessary, and loads the model before opening an interactive chat or processing a single prompt.

Open an interactive chat:

```bash
llamapi run qwen3:4b
```

Send a single prompt:

```bash
llamapi run qwen3:4b "Explain what an NPU is."
```

Set a system prompt and open an interactive chat:

```bash
llamapi run qwen3:4b --system "You are an embedded Linux engineer."
```

When `run` exits, it automatically unloads a model newly loaded by that command, so the model can no longer receive API requests. Use `load` if the model must remain available.

### Use Interactive Mode

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

## Deploy a Model (`load`)

The `load` command loads a downloaded model into the LlamaPi service. The command finishes after loading completes, and closing the terminal does not affect the deployed model.

Before using `load`, download the model and make sure the LlamaPi service is running. If the service is stopped, run:

```bash
sudo systemctl start llamapi-server
```

Load and deploy one model instance:

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

### List Deployed Models

```bash
llamapi ps
```

This command displays each model's runtime ID, type, inference platform, state, instance count, and automatic-loading state.

### Resize Instances

Resize a deployed model to three instances:

```bash
llamapi load assistant --instance 3
```

More instances can increase parallel request capacity but consume more hardware resources. Select a count appropriate for the device.

### Unload a Model

```bash
llamapi unload assistant
```

Unloading ends the model deployment and releases its runtime resources, but it does not remove the local model files.

### Use an Embedding Model

Embedding models cannot be used for chat through `llamapi run`. Download and deploy the model first:

```bash
llamapi pull bge-m3
llamapi load bge-m3
llamapi ps
```

Then call `/v1/embeddings`. See the [API Reference](../advanced-guides/api-reference.md#embeddings) for request details.

A deployment created with `load` lasts only while the current LlamaPi service is running. To load a model automatically when the service starts, continue with [Persistent Deployment](./model-persistence.md).
