# Changelog

This page records LlamaPi features, behavior changes, and fixes.

`llamapi-cli` and `llamapi-server` always use the same version number.

## 0.2.0

This is the first LlamaPi release.

### Added

#### Model Deployment and Execution

- Added model discovery, download, loading, execution, and unloading.
- Added interactive chat, single prompts, system prompts, generation-parameter updates, and multiline input to the `llamapi-cli`.
- Added runtime model IDs, multiple instances, instance resizing, and request queues.

#### Model Management

- Added local-model, loaded-model, and runtime-state inspection.
- Added removal of local model files.
- Added `enable` and `disable` commands for automatic model loading.

#### Platform and Model Support

- Added automatic model-variant selection based on current hardware and local model availability.
- Added `rknn3` and `rkllm` chat platforms and the `rknn2` Embedding platform.
- Added Hugging Face and ModelScope sources with concurrent automatic probing.

#### API Services

- Added OpenAI-compatible Chat Completions, Embeddings, and Models APIs.
- Added extension APIs for model loading, resizing, unloading, and platform discovery.
- Added regular JSON and SSE streaming responses for Chat Completions.
- Added protocol structures for text, image, audio, and function-tool content.

#### Configuration and Operations

- Added the `llamapi-server` systemd service and TOML configuration.
- Added a unified `9265` service-port configuration.

### Changed

- None.

### Fixed

- None.
