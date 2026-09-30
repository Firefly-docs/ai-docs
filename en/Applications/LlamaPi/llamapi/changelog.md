# Changelog

This page records LlamaPi features, behavior changes, and fixes.

`llamapi-cli` and `llamapi-server` always use the same version number.

## 0.3.2

- Model discovery, download, loading, execution, and unloading; downloads support resuming.
- `llamapi-cli` supports interactive chat, single prompts, multiline input, and multimodal attachments.
- Multiple instances, instance resizing, and automatic model-loading configuration.
- `rknn3` and `rkllm` chat platforms and the `rknn2` Embedding platform; multi-chip models are supported.
- OpenAI-compatible Chat Completions, Embeddings, and Models APIs, plus model-management extension APIs.
- `llamapi-server` and `llamapi-modelstore` systemd services, and a Windows client that runs on the DirectAI platform.
