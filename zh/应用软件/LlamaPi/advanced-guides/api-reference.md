# API 接口详解

`llamapi-server` 提供 OpenAI 兼容的对话、Embedding 和模型查询接口，以及用于端侧模型实例管理和硬件平台查询的 LlamaPi 扩展接口。

## 接口概览

| 项目 | 说明 |
|:---:|:---:|
| Base URL | `http://127.0.0.1:9265` |
| API 前缀 | OpenAI 兼容接口和管理接口使用 `/v1`；健康检查除外 |
| 请求格式 | `POST` 使用 JSON body；`GET` 不需要 body |
| 响应格式 | 普通接口返回 JSON；流式对话返回 SSE；健康检查返回纯文本 |
| 请求体上限 | 64 MiB |
| 鉴权 | 无鉴权要求 |
| CORS | 允许跨域请求 |

当前服务默认不要求鉴权，并允许跨域访问。将服务暴露到不可信网络前，应增加防火墙、反向代理或其他访问控制。

## 快速调用

加载模型：

```bash
curl -s http://127.0.0.1:9265/v1/models/load \
  -H 'Content-Type: application/json' \
  -d '{
    "model_id": "Qwen3",
    "model_path": "/var/lib/llamapi/models/rkllm/rk3588/qwen3-4b"
  }'
```

调用对话接口：

```bash
curl -s http://127.0.0.1:9265/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "Qwen3",
    "messages": [
      { "role": "user", "content": "你好" }
    ]
  }'
```

## 通用约定

| 项目 | 说明 |
|:---:|:---:|
| 模型 ID | `model` 或 `model_id` 必须对应已加载模型 |
| 流式响应 | 对话请求设置 `"stream": true` 后返回 SSE，结束时发送 `data: [DONE]` |
| 多模态输入 | 协议可接收 `text`、`image_url` 和 `input_audio` |
| 错误响应 | 普通错误使用 OpenAI 风格 `{ "error": ... }` JSON |
| SSE 错误 | 流式生成中途失败时发送 `event: error` |

客户端应通过 `/v1/models` 和 `/v1/platforms` 发现模型与平台，不要写死运行时模型 ID、平台名称、芯片型号或模型路径。

## 数据取值约定

### 闭合集合

以下字段可以按固定集合处理：

| 字段 | 取值 | 说明 |
|:---:|:---:|:---:|
| `messages[].role` | `system`、`user`、`assistant`、`tool` | 对话消息角色 |
| `messages[].content[].type` | `text`、`image_url`、`input_audio` | 支持的 content part |
| `image_url.url` 的 data URL 格式 | `png`、`jpeg`、`jpg`、`webp` | 本地路径不使用该枚举 |
| `input_audio.format` | `wav`、`mp3` | 具体模型可能只支持其中一部分 |
| `encoding_format` | `float`、`base64` | Embedding 输出编码 |
| `model_kind` | `chat`、`embedding` | 模型能力类型 |
| `error.type` | `invalid_request_error`、`rate_limit_exceeded`、`server_error` | 错误类型 |

### 已知但可扩展的响应值

| 字段 | 当前已知值 | 客户端处理建议 |
|:---:|:---:|:---:|
| `choices[].finish_reason` | `stop`、`length`、`tool_calls` | 兼容未来新增字符串 |
| `object` | `chat.completion`、`chat.completion.chunk`、`list`、`embedding`、`model` | 根据接口和字段结构处理 |
| `error.code` | 见[错误响应](#错误响应) | 未知值按通用错误处理 |

### 开放值

以下字段不应当作为固定枚举：

| 字段 | 说明 |
|:---:|:---:|
| `model`、`model_id` | 由加载模型时的 ID 决定 |
| `id` | `llamapi-server` 生成的响应 ID |
| `platform` | 通过平台接口发现 |
| `owned_by` | 当前形式为 `llamapi/{platform}`，客户端不应依赖固定格式 |
| `model_path` | `llamapi-server` 文件系统路径 |
| `detected_chips[].chip_type` | 当前主机检测到的芯片型号 |
| `message` | 人类可读文本，不建议程序解析 |
| `tool_call_id`、`tool_calls[].id` | 由模型输出或请求上下文决定 |
| `tools[].type` | 建议使用 `function`，但服务端按字符串解析 |
| `tools[].function.name` | 由客户端定义 |

## 接口列表

| Method | Path | 类型 | 说明 |
|:---:|:---:|:---:|:---:|
| `POST` | `/v1/chat/completions` | OpenAI 兼容 | 对话补全，支持 JSON 和 SSE |
| `POST` | `/v1/embeddings` | OpenAI 兼容 | 文本向量，支持单条和批量输入 |
| `GET` | `/v1/models` | OpenAI 兼容 | 列出已加载模型 |
| `GET` | `/v1/models/{model_id}` | OpenAI 兼容 | 查询单个已加载模型 |
| `POST` | `/v1/models/load` | LlamaPi 扩展 | 动态加载模型 |
| `POST` | `/v1/models/resize` | LlamaPi 扩展 | 调整模型实例数 |
| `POST` | `/v1/models/unload` | LlamaPi 扩展 | 卸载模型 |
| `GET` | `/v1/platforms` | LlamaPi 扩展 | 查询平台和检测到的芯片 |
| `GET` | `/health` | 健康检查 | 检查 HTTP 服务是否运行 |

## 错误响应

应用错误使用 OpenAI 风格结构：

```json
{
  "error": {
    "message": "Model 'demo' not found",
    "type": "invalid_request_error",
    "param": "model",
    "code": "model_not_found"
  }
}
```

| HTTP 状态 | `type` | `code` | 触发条件 |
|:---:|:---:|:---:|:---:|
| `400` | `invalid_request_error` | `unsupported_platform` | 模型平台不受支持 |
| `400` | `invalid_request_error` | `platform_detection_failed` | 无法从模型目录识别平台 |
| `400` | `invalid_request_error` | `wrong_model_type` | 对话接口调用 Embedding 模型，或反过来 |
| `400` | `invalid_request_error` | `unsupported_content_part_type` | 包含不支持的 content part |
| `400` | `invalid_request_error` | `invalid_content_part` | 图片、音频等 content part 格式非法 |
| `400` | `invalid_request_error` | `context_length_exceeded` | 输入超过模型上下文上限 |
| `400` | `invalid_request_error` | `invalid_instance_count` | 实例数为 `0` |
| `404` | `invalid_request_error` | `model_not_found` | 模型未加载 |
| `409` | `invalid_request_error` | `model_already_exists` | 模型 ID 已存在 |
| `429` | `rate_limit_exceeded` | `queue_full` | 模型实例和等待队列都已满 |
| `500` | `server_error` | `null` | 引擎、配置或内部错误 |

负数实例数会在 JSON 解析阶段失败。

## Chat Completions

```text
POST /v1/chat/completions
```

对话补全接口。`stream` 缺省为 `false`，模型必须是 `chat` 类型。

### 请求字段

| 字段 | 类型 | 必填 | 说明 |
|:---:|:---:|:---:|:---:|
| `model` | string | 是 | 已加载模型 ID |
| `messages` | array | 是 | 对话消息数组 |
| `stream` | boolean | 否 | `true` 时返回 SSE |
| `temperature` | number | 否 | 覆盖默认采样温度 |
| `top_p` | number | 否 | 覆盖默认 top-p |
| `top_k` | integer | 否 | 覆盖默认 top-k |
| `repeat_penalty` | number | 否 | 覆盖默认重复惩罚 |
| `frequency_penalty` | number | 否 | 覆盖默认频率惩罚 |
| `presence_penalty` | number | 否 | 覆盖默认存在惩罚 |
| `max_tokens` | integer | 否 | 最大生成 token 数 |
| `max_completion_tokens` | integer | 否 | 与 `max_tokens` 等价；两者同时传入时优先使用本字段 |
| `stop` | string array | 否 | 停止序列 |
| `tools` | array | 否 | OpenAI 风格 function tools |
| `tool_choice` | string | 否 | 会被解析，但当前 `llamapi-server` 不使用该值 |
| `enable_thinking` | boolean | 否 | 覆盖模型思考模式配置 |

### 消息字段

`messages[]` 支持：

| 字段 | 类型 | 必填 | 说明 |
|:---:|:---:|:---:|:---:|
| `role` | string | 是 | `system`、`user`、`assistant` 或 `tool` |
| `content` | string 或 array | 否 | 纯文本或 content parts |
| `tool_call_id` | string | 否 | tool 消息对应的工具调用 ID |
| `tool_calls` | array | 否 | assistant 消息中的工具调用 |

### 多模态内容

当 `content` 是数组时，支持：

文本：

```json
{
  "type": "text",
  "text": "描述这张图片"
}
```

图片：

```json
{
  "type": "image_url",
  "image_url": {
    "url": "/path/to/image.jpg"
  }
}
```

图片 URL 支持：

- `llamapi-server` 本地图片路径。
- `png`、`jpeg`、`jpg`、`webp` 的 base64 data URL。

不支持 `http://` 或 `https://` 远程图片 URL。

音频：

```json
{
  "type": "input_audio",
  "input_audio": {
    "format": "wav",
    "data": "<base64>"
  }
}
```

协议层支持 `wav` 和 `mp3`，具体模型可能只支持其中一部分。

`video`、`file` 和未知类型会返回 `unsupported_content_part_type`。

### 工具定义

`tools[]` 使用 OpenAI function tool 结构：

| 字段 | 类型 | 必填 |
|:---:|:---:|:---:|
| `type` | string | 是，建议为 `function` |
| `function.name` | string | 是 |
| `function.description` | string | 是 |
| `function.parameters` | JSON | 是 |

### 非流式请求

```bash
curl -s http://127.0.0.1:9265/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "Qwen3",
    "messages": [
      { "role": "user", "content": "用一句话介绍 LlamaPi。" }
    ],
    "max_tokens": 128
  }'
```

### 非流式响应

```json
{
  "id": "chatcmpl-...",
  "object": "chat.completion",
  "created": 1710000000,
  "model": "Qwen3",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "...",
        "tool_calls": []
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 13,
    "completion_tokens": 14,
    "total_tokens": 27
  }
}
```

### 流式请求

```bash
curl -N http://127.0.0.1:9265/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "Qwen3",
    "messages": [
      { "role": "user", "content": "你好" }
    ],
    "stream": true
  }'
```

### 流式响应

每个 SSE 事件的 `data` 是 `chat.completion.chunk` JSON。生成结束后，`llamapi-server` 发送独立的 usage chunk，再发送 `[DONE]`：

```text
data: {"id":"chatcmpl-...","object":"chat.completion.chunk","created":1710000000,"model":"Qwen3","choices":[{"index":0,"delta":{"role":"assistant","content":"你"}}]}

data: {"id":"chatcmpl-...","object":"chat.completion.chunk","created":1710000000,"model":"Qwen3","choices":[{"index":0,"delta":{"content":"好"}}]}

data: {"id":"chatcmpl-...","object":"chat.completion.chunk","created":1710000000,"model":"Qwen3","choices":[{"index":0,"delta":{},"finish_reason":"stop"}]}

data: {"id":"chatcmpl-...","object":"chat.completion.chunk","created":1710000000,"model":"Qwen3","choices":[],"usage":{"prompt_tokens":13,"completion_tokens":14,"total_tokens":27}}

data: [DONE]
```

流式生成中途失败时，`llamapi-server` 发送：

```text
event: error
data: {"error":{...}}
```

错误后终止流，不再发送 `[DONE]`。

## Embeddings

```text
POST /v1/embeddings
```

模型必须是 `embedding` 类型。

### Embeddings 请求字段

| 字段 | 类型 | 必填 | 说明 |
|:---:|:---:|:---:|:---:|
| `model` | string | 是 | 已加载 Embedding 模型 ID |
| `input` | string 或 string array | 是 | 单条或批量文本 |
| `encoding_format` | string | 否 | `float` 或 `base64`；默认 `float` |
| `dimensions` | integer | 否 | 会被解析，但当前 `llamapi-server` 不裁剪向量 |
| `user` | string | 否 | 会被解析，但当前 `llamapi-server` 不使用 |

`base64` 表示 little-endian `f32` 原始字节的 base64 编码。

### 加载 Embedding 模型

可以使用 `llamapi-cli`：

```bash
llamapi load bge-m3
llamapi ps
```

也可以调用管理 API：

```bash
curl -s http://127.0.0.1:9265/v1/models/load \
  -H 'Content-Type: application/json' \
  -d '{
    "model_id": "bge-m3",
    "model_path": "/var/lib/llamapi/models/rknn2/rk3588/bge-m3"
  }'
```

### Embeddings 请求示例

```bash
curl -s http://127.0.0.1:9265/v1/embeddings \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "bge-m3",
    "input": ["hello", "LlamaPi"],
    "encoding_format": "float"
  }'
```

### Embeddings 响应示例

```json
{
  "object": "list",
  "data": [
    {
      "object": "embedding",
      "embedding": [0.1, 0.2],
      "index": 0
    }
  ],
  "model": "bge-m3",
  "usage": {
    "prompt_tokens": 2,
    "total_tokens": 2
  }
}
```

## 查询模型

### 列出模型

```text
GET /v1/models
```

```bash
curl -s http://127.0.0.1:9265/v1/models
```

响应：

```json
{
  "object": "list",
  "data": [
    {
      "id": "Qwen3",
      "object": "model",
      "created": 0,
      "owned_by": "llamapi/rkllm",
      "platform": "rkllm",
      "instance_count": 1,
      "model_path": "/var/lib/llamapi/models/rkllm/rk3588/qwen3-4b",
      "model_kind": "chat"
    }
  ]
}
```

### 查询单个模型

```text
GET /v1/models/{model_id}
```

```bash
curl -s http://127.0.0.1:9265/v1/models/Qwen3
```

响应对象字段与 `/v1/models` 的 `data[]` 相同。模型不存在时返回 `404 model_not_found`。

## 加载模型

```text
POST /v1/models/load
```

### 加载模型请求字段

| 字段 | 类型 | 必填 | 说明 |
|:---:|:---:|:---:|:---:|
| `model_id` | string | 是 | 对外暴露的模型 ID |
| `model_path` | string | 是 | `llamapi-server` 文件系统中的模型目录 |
| `instance_count` | integer | 否 | 目标实例数，默认 `1`，必须大于等于 `1` |
| `request_queue_size` | integer | 否 | 模型等待队列容量；缺省使用 `llamapi-server` 配置 |
| `default_params` | object | 否 | 模型默认生成参数 |

`default_params` 支持：

- `temperature`
- `top_p`
- `top_k`
- `repeat_penalty`
- `frequency_penalty`
- `presence_penalty`
- `max_tokens`
- `max_context_len`
- `stop`
- `enable_thinking`

### 加载模型请求示例

```bash
curl -s http://127.0.0.1:9265/v1/models/load \
  -H 'Content-Type: application/json' \
  -d '{
    "model_id": "Qwen3",
    "model_path": "/var/lib/llamapi/models/rkllm/rk3588/qwen3-4b",
    "instance_count": 2,
    "default_params": {
      "temperature": 0.7,
      "max_tokens": 512
    }
  }'
```

### 加载模型响应示例

```json
{
  "success": true,
  "message": "model 'Qwen3' loaded",
  "requested_instance_count": 2,
  "actual_instance_count": 2
}
```

如果至少一个实例加载成功，接口返回 `200 OK`：

- 全部成功时，`message` 为 `model '<id>' loaded`。
- 部分成功时，`message` 为 `model '<id>' partially loaded`。
- `requested_instance_count` 是目标数量。
- `actual_instance_count` 是实际加载数量。

### 协处理器实例限制

在协处理器芯片上通过 `/v1/models/load` 或 `/v1/models/resize` 请求多个模型实例，可能导致加载失败、芯片通信失败和服务异常。当前应将协处理器模型的 `instance_count` 保持为 `1`。故障恢复步骤见[协处理器加载多个模型实例后通信失败](../llamapi/faq.md#协处理器加载多个模型实例后通信失败)。

## 调整实例数

```text
POST /v1/models/resize
```

| 字段 | 类型 | 必填 | 说明 |
|:---:|:---:|:---:|:---:|
| `model_id` | string | 是 | 已加载模型 ID |
| `instance_count` | integer | 是 | 目标实例数，必须大于等于 `1` |

```bash
curl -s http://127.0.0.1:9265/v1/models/resize \
  -H 'Content-Type: application/json' \
  -d '{
    "model_id": "Qwen3",
    "instance_count": 1
  }'
```

响应：

```json
{
  "success": true,
  "message": "model 'Qwen3' resized",
  "requested_instance_count": 1,
  "actual_instance_count": 1
}
```

扩容只成功创建部分实例时仍可能返回 `200 OK`，`message` 为 `model '<id>' partially resized`。

## 卸载模型

```text
POST /v1/models/unload
```

| 字段 | 类型 | 必填 | 说明 |
|:---:|:---:|:---:|:---:|
| `model_id` | string | 是 | 已加载模型 ID |

```bash
curl -s http://127.0.0.1:9265/v1/models/unload \
  -H 'Content-Type: application/json' \
  -d '{ "model_id": "Qwen3" }'
```

响应：

```json
{
  "success": true,
  "message": "model 'Qwen3' unloaded"
}
```

## 查询平台

```text
GET /v1/platforms
```

返回已注册的对话与 Embedding 平台，以及当前主机检测到的芯片。

```bash
curl -s http://127.0.0.1:9265/v1/platforms
```

响应示例：

```json
{
  "platforms": [
    {
      "id": "rknn3",
      "display_name": "RKNN3",
      "available": true,
      "detected_chips": [
        {
          "chip_type": "RK1828",
          "count": 2
        }
      ]
    }
  ]
}
```

`detected_chips` 按芯片型号汇总数量。没有检测到芯片时返回空数组。

## 健康检查

```text
GET /health
```

```bash
curl -s http://127.0.0.1:9265/health
```

响应：

```text
ok
```

健康接口只检查 HTTP 服务是否运行，不代表已经加载模型。模型可用性应通过 `/v1/models` 和实际推理请求确认。
