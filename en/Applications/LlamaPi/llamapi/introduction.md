# Introduction

LlamaPi (pronounced “Llama Pie”) is a large-model deployment and management tool developed by the Firefly AI team. It is designed for edge devices with large-model inference capabilities, helping users deploy and manage open-source models locally and provide inference to third-party applications through OpenAI-compatible APIs.

LlamaPi lowers the barrier to deploying large models on edge devices. It shortens the workflow from obtaining a model and adapting it to the hardware to running the model, providing a lightweight foundation for offline and private AI applications.

Ready to try LlamaPi? Follow the [Quick Start](../getting-started/quickstart.md) to install it and run your first model.

## What Is LlamaPi?

Running a large model on an edge-computing device typically involves hardware detection, inference-platform selection, model-format matching, runtime configuration, and service deployment. LlamaPi brings these steps together behind a consistent command-line workflow and service API so users can download, run, manage, and integrate models with fewer manual steps.

LlamaPi is not a model or a single inference engine. It is a deployment and management tool that connects devices, models, on-device inference platforms, and applications.

## Problems LlamaPi Solves

- **Complex deployment workflows**: combines model discovery, download, loading, and execution into a continuous workflow.
- **High hardware-adaptation overhead**: filters compatible models based on device detection and available inference platforms.
- **Fragmented model management**: provides consistent commands for local models, runtime state, instance counts, and automatic loading.
- **Application-integration cost**: provides local inference to third-party applications through OpenAI-compatible APIs.

## Core Capabilities

- **Deploy open-source models locally**: download, load, and run compatible chat and Embedding models on the device.
- **Manage models consistently**: discover models, inspect runtime state, resize instances, configure automatic loading, unload models, and remove local files.
- **Support multiple inference platforms**: detect inference platforms available on the current device and find compatible models.
- **Provide OpenAI-compatible APIs**: currently supports Chat Completions, Embeddings, and Models APIs.
- **Provide LlamaPi extensions**: manage model instances and query hardware platforms through additional endpoints.

## Typical Use Cases

- Provide offline, private AI inference where internet access is unavailable or data must remain local.
- Build local question-answering, text-generation, and assistant applications on development boards or edge devices.
- Supply local model services to third-party applications that support a custom OpenAI service URL.
- Provide local Embedding services for retrieval, search, or knowledge-base applications.

## Components

LlamaPi has two main components:

| Component | Purpose |
|:---:|:---:|
| `llamapi-cli` | Command-line component for discovering, downloading, running, and managing models; invoked with the `llamapi` command |
| `llamapi-server` | Service component that detects inference platforms, loads models, manages model instances, and provides HTTP APIs |

`llamapi-cli` provides convenient interactive operations and common workflows, while `llamapi-server` hosts persistent model services. Together they cover the path from obtaining a model to connecting an application.

## Supported Scope and Current Limitations

- LlamaPi targets edge devices with large-model inference capabilities. See [Download and Manage Models](../getting-started/model-download-and-management.md) for currently supported hardware and inference platforms.
- The current API documentation covers OpenAI-compatible Chat Completions, Embeddings, and Models APIs, plus LlamaPi-specific extensions. This does not mean that every OpenAI endpoint and field is supported.
- The current version does not provide Anthropic-compatible APIs.
- Whether a model can run depends on the device hardware, available inference platform, and model variant.

## Documentation Guide

- Set up LlamaPi for the first time: [Quick Start](../getting-started/quickstart.md)
- Download and manage local models: [Download and Manage Models](../getting-started/model-download-and-management.md)
- Load, run, and unload models: [Load and Run Models](../getting-started/model-load-and-run.md)
- Configure automatic model loading: [Persistent Deployment](../getting-started/model-persistence.md)
- Connect an existing application to a local model: [Connect Third-Party Applications](../getting-started/third-party-integration.md)
- Look up complete command options: [Terminal Command Guide](../advanced-guides/cli-command-guide.md)
- Configure and operate the service: [Service Configuration and Operations](../advanced-guides/server-operations.md)
- Build an API integration: [API Reference](../advanced-guides/api-reference.md)
