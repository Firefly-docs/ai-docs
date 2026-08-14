# 更新日志

本页面记录 LlamaPi 的功能、行为变更和修复。

`llamapi-cli` 与 `llamapi-server` 始终使用相同版本号。

## 0.2.0

这是 LlamaPi 的首个发布版本。

### 新增

#### 模型部署与运行

- 提供模型查找、下载、加载、运行和卸载能力。
- `llamapi-cli` 支持交互式对话、单次提问、系统提示词、生成参数调整和多行输入。
- 支持模型运行时 ID、多实例加载、实例调整和请求等待队列。

#### 模型管理

- 支持查看本地模型、已加载模型和模型运行状态。
- 支持删除本地模型文件。
- 支持通过 `enable` 和 `disable` 配置模型自动加载。

#### 平台与模型支持

- 支持根据当前硬件和本地模型状态自动选择模型变体。
- 支持 `rknn3`、`rkllm` 对话模型平台和 `rknn2` Embedding 模型平台。
- 支持 Hugging Face 和 ModelScope 模型源，以及自动并发探测。

#### API 服务

- 提供 OpenAI 兼容的 Chat Completions、Embeddings 和 Models 接口。
- 提供模型加载、实例调整、模型卸载和平台查询扩展接口。
- Chat Completions 支持普通 JSON 响应和 SSE 流式响应。
- 对话协议支持文本、图片、音频和 function tools 数据结构。

#### 配置与运维

- 提供 `llamapi-server` systemd 服务和 TOML 配置。
- 提供统一的 `9265` 服务端口配置。

### 变更

- 无。

### 修复

- 无。
