# 更新日志

本页面记录 LlamaPi 的功能、行为变更和修复。

`llamapi-cli` 与 `llamapi-server` 始终使用相同版本号。

## 0.3.2

- 提供模型查找、下载、加载、运行和卸载能力，下载支持断点续传。
- `llamapi-cli` 支持交互式对话、单次提问、多行输入和多模态附件。
- 支持多实例加载、实例调整和模型自动加载配置。
- 支持 `rknn3`、`rkllm` 对话模型平台和 `rknn2` Embedding 模型平台，支持多芯片协同模型。
- 提供 OpenAI 兼容的 Chat Completions、Embeddings 和 Models 接口，以及模型管理扩展接口。
- 提供 `llamapi-server`、`llamapi-modelstore` systemd 服务和运行在 DirectAI 平台上的 Windows 客户端。
