# 服务配置与运维

`llamapi-server` 是 LlamaPi 的服务端组件，负责检测推理平台、加载模型、管理模型实例，并提供 OpenAI 兼容的 HTTP API。

本章假设已经按照[快速开始](../getting-started/quickstart.md)安装 `firefly-llamapi-server`，重点介绍服务管理、配置、日志、模型预加载和故障排查。

## 服务信息

| 项目 | 值 |
|:---:|:---:|
| Debian 包名 | `firefly-llamapi-server` |
| systemd unit | `llamapi-server.service` |
| 二进制路径 | `/usr/bin/llamapi-server` |
| 默认配置文件 | `/etc/llamapi-server/config.toml` |
| 默认监听地址 | `0.0.0.0:9265` |
| 健康检查 | `GET /health` |

systemd 使用以下形式启动服务：

```text
ExecStart=/usr/bin/llamapi-server --config /etc/llamapi-server/config.toml
```

## 管理服务

| 操作 | 命令 |
|:---:|:---:|
| 查看状态 | `systemctl status llamapi-server` |
| 启动 | `sudo systemctl start llamapi-server` |
| 停止 | `sudo systemctl stop llamapi-server` |
| 重启 | `sudo systemctl restart llamapi-server` |
| 设置开机启动 | `sudo systemctl enable llamapi-server` |
| 取消开机启动 | `sudo systemctl disable llamapi-server` |
| 重载 systemd unit | `sudo systemctl daemon-reload` |

修改配置文件后需要重启服务：

```bash
sudo systemctl restart llamapi-server
```

## 查看日志

实时查看日志：

```bash
journalctl -u llamapi-server -f
```

查看本次系统启动后的服务日志：

```bash
journalctl -u llamapi-server -b
```

查看最近 100 行：

```bash
journalctl -u llamapi-server -n 100
```

排查启动、配置解析、平台加载和模型预加载问题时，应优先检查服务日志。

## 启动参数

`llamapi-server` 支持：

| 参数 | 说明 |
|:---:|:---:|
| `--host <HOST>` | 覆盖监听地址 |
| `-p, --port <PORT>` | 覆盖监听端口 |
| `-c, --config <CONFIG>` | 指定 TOML 配置文件 |
| `--log-level <LEVEL>` | 设置 `trace`、`debug`、`info`、`warn` 或 `error` |
| `-h, --help` | 显示帮助 |
| `-V, --version` | 显示版本、commit hash 和构建时间 |

配置优先级从高到低为：

1. 命令行参数。
2. 配置文件。
3. 程序内置默认值。

systemd 服务默认通过配置文件启动。除临时调试外，建议在 `/etc/llamapi-server/config.toml` 中维护长期配置。

## 配置文件

默认路径：

```text
/etc/llamapi-server/config.toml
```

基础配置：

```toml
[server]
host = "0.0.0.0"
port = 9265
log_level = "info"
```

包含全局生成参数和模型预加载的完整示例：

```toml
[server]
host = "0.0.0.0"
port = 9265
log_level = "info"
request_queue_size = 48

[defaults]
temperature = 1.0
top_p = 0.9
top_k = 1
repeat_penalty = 1.2
frequency_penalty = 0.0
presence_penalty = 0.0
max_tokens = 1024
max_context_len = 4096
stop = []
enable_thinking = false

[[models]]
model_id = "qwen3:4b@rkllm-rk3588"
model_path = "/var/lib/llamapi/models/rkllm/rk3588/qwen3-4b"
instance_count = 1
request_queue_size = 48

[models.default_params]
temperature = 0.7
top_p = 0.9
max_tokens = 1024
max_context_len = 4096
stop = []
enable_thinking = false
```

### `[server]` 配置

| 字段 | 默认值 | 说明 |
|:---:|:---:|:---:|
| `host` | `0.0.0.0` | HTTP 监听地址 |
| `port` | `9265` | HTTP 监听端口 |
| `log_level` | `info` | 日志级别 |
| `request_queue_size` | `48` | 每个模型默认等待容量 |

模型的总接受容量约为：

```text
实际实例数 + request_queue_size
```

实例都在处理请求且等待队列已满时，`llamapi-server` 返回 HTTP `429` 和错误码 `queue_full`。

### `[defaults]` 配置

`[defaults]` 为所有模型提供全局生成参数。模型目录中的 `model.toml` 和 `[[models]].default_params` 可以覆盖这些值。

| 字段 | 说明 |
|:---:|:---:|
| `temperature` | 采样温度 |
| `top_p` | nucleus sampling 参数 |
| `top_k` | top-k 采样参数 |
| `repeat_penalty` | 重复惩罚 |
| `frequency_penalty` | 频率惩罚 |
| `presence_penalty` | 存在惩罚 |
| `max_tokens` | 默认最大生成 token 数 |
| `max_context_len` | 运行时上下文上限 |
| `stop` | 停止序列数组 |
| `enable_thinking` | 是否启用模型思考模式 |

请求中的同名字段可以覆盖模型默认参数。具体字段见[API 接口详解](./api-reference.md#请求字段)。

### `[[models]]` 配置

每个 `[[models]]` 条目定义一个随 `llamapi-server` 启动预加载的模型组。

| 字段 | 必填 | 说明 |
|:---:|:---:|:---:|
| `model_id` | 是 | 对外暴露的运行时模型 ID |
| `model_path` | 是 | 模型目录路径 |
| `instance_count` | 否 | 目标实例数，默认 `1`，必须大于等于 `1` |
| `request_queue_size` | 否 | 该模型的等待容量；缺省时使用 `llamapi-server` 默认值 |
| `default_params` | 否 | 该模型的默认生成参数 |

可以使用 `llamapi-cli` 管理这些条目：

```bash
llamapi enable qwen3:4b --instance 2
llamapi disable qwen3:4b
```

建议优先使用 `llamapi-cli` 修改自动加载配置，避免手动生成错误的模型路径或运行时 ID。

**协处理器限制**：协处理器模型的 `instance_count` 当前应设置为 `1`。配置多个实例可能导致模型加载失败、协处理器通信失败以及 `rknn3.service` 和 `llamapi-server.service` 异常。完整恢复步骤见[协处理器加载多个模型实例后通信失败](../llamapi/faq.md#协处理器加载多个模型实例后通信失败)。

## 模型预加载

服务启动时按顺序读取 `[[models]]` 并加载模型。

- 单个模型预加载失败会写入错误日志。
- 单个模型失败不会阻止 HTTP 服务继续启动。
- 请求多个实例时，实际成功数量可能少于目标数量。
- `/v1/models` 中的 `instance_count` 表示当前实际可服务实例数。

确认预加载结果：

```bash
curl -s http://127.0.0.1:9265/v1/models
```

响应中的重要字段：

| 字段 | 说明 |
|:---:|:---:|
| `id` | 对外使用的模型 ID |
| `platform` | 推理平台 |
| `instance_count` | 实际活动实例数 |
| `model_path` | 模型加载目录 |
| `model_kind` | `chat` 或 `embedding` |

## 运行时模型管理

除启动预加载外，也可以在运行期间调用管理 API：

| 操作 | API | `llamapi-cli` 对应命令 |
|:---:|:---:|:---:|
| 加载模型 | `POST /v1/models/load` | `llamapi load` |
| 调整实例数 | `POST /v1/models/resize` | `llamapi load <id> --instance <N>` |
| 卸载模型 | `POST /v1/models/unload` | `llamapi unload` |

使用 `llamapi-cli` 时，模型命名、路径解析和运行时 ID 由客户端处理。直接调用 API 时，需要提供 `llamapi-server` 文件系统中的完整模型路径。

## 健康检查

健康接口不依赖是否已经加载模型：

```bash
curl -s http://127.0.0.1:9265/health
```

返回：

```text
ok
```

`ok` 只表示 HTTP 服务正在运行。要确认模型可以处理请求，还需要检查：

```bash
curl -s http://127.0.0.1:9265/v1/models
curl -s http://127.0.0.1:9265/v1/platforms
```

## 配置安全与网络访问

当前 `llamapi-server`：

- 不要求 API 鉴权。
- 允许跨域请求。
- 默认监听 `0.0.0.0:9265`。

因此，将服务暴露到不可信网络前，应通过防火墙、反向代理或受控网络限制访问范围。

## 故障排查

| 现象 | 检查项 | 处理方式 |
|:---:|:---:|:---:|
| `/health` 连接失败 | 服务状态、监听端口 | 执行 `systemctl status llamapi-server`，确认配置端口为 `9265` |
| 服务启动失败 | TOML 是否可解析 | 查看 `journalctl -u llamapi-server -b` 中的 `failed to parse config file` |
| 平台显示不可用 | 后端运行库和硬件环境 | 确认对应运行库已安装，并检查 Debian 包是否包含所需动态库 |
| 模型预加载失败 | `model_path`、`model.toml`、模型文件 | 查看日志中的 `Failed to preload model` |
| 协处理器加载多个模型实例后通信失败 | 协处理器是否配置了多个实例 | 停止继续加载模型，并按[故障恢复步骤](../llamapi/faq.md#协处理器加载多个模型实例后通信失败)依次重置芯片和重启服务 |
| `model_not_found` | 模型是否已加载、ID 是否一致 | 调用 `/v1/models` 或执行 `llamapi ps` |
| `queue_full` | 实例都忙且队列已满 | 增加实例数或 `request_queue_size`，或降低并发 |
| `wrong_model_type` | API 与模型类型不匹配 | 对话接口使用 `chat` 模型，Embedding 接口使用 `embedding` 模型 |
| `invalid_instance_count` | 实例数为 `0` | 将实例数改为 `1` 或更大 |
| 监听失败 | 端口被占用 | 停止占用 `9265` 的进程，或在部署环境中协调其他服务端口 |

更多跨 `llamapi-cli`、`llamapi-server` 和 API 的问题见[常见问题与故障排查](../llamapi/faq.md)。
