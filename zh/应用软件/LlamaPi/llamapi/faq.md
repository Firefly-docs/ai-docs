# 常见问题与故障排查

本章汇总 `llamapi-cli`、`llamapi-server` 和 API 使用过程中的常见问题。排查前建议先运行以下命令：

```bash
llamapi platform                         # 检查硬件平台和服务状态
llamapi ps                               # 查看已加载模型和自动加载配置
systemctl status llamapi-server          # 查看 llamapi-server 服务状态
curl -s http://127.0.0.1:9265/health     # 检查 API 健康状态
```

## 快速定位

| 问题现象 | 排查分类 |
|:---:|:---:|
| `llamapi-cli` 与 `llamapi-server` 版本不一致 | [版本与软件包](#版本与软件包) |
| 服务未运行、健康检查失败或未检测到可用推理平台 | [服务状态与平台检测](#服务状态与平台检测) |
| 模型列表、下载源或平台变体异常 | [模型查询、下载与选择](#模型查询下载与选择) |
| 模型加载失败、实例异常、运行时清理问题 | [模型加载与实例管理](#模型加载与实例管理) |
| API 模型 ID、模型类型、上下文或多模态输入错误 | [HTTP API 请求](#http-api-请求) |
| 需要收集日志和环境信息 | [诊断信息](#诊断信息) |

## 版本与软件包

### 检查 `llamapi-cli` 与 `llamapi-server` 版本一致性

检查 `llamapi-cli`：

```bash
llamapi --version
```

检查 Debian 软件包：

```bash
dpkg-query -W -f='${Package}: ${Version}\n' \
  firefly-llamapi-cli firefly-llamapi-server
```

`firefly-llamapi-cli` 和 `firefly-llamapi-server` 的版本号应保持一致。

## 服务状态与平台检测

### `llamapi platform` 显示服务未运行

检查服务：

```bash
systemctl status llamapi-server
```

启动服务：

```bash
sudo systemctl start llamapi-server
```

然后确认健康接口：

```bash
curl -s http://127.0.0.1:9265/health
```

如果服务实际已经运行，但 `llamapi-cli` 仍显示未运行，可能是系统中的 HTTP 代理拦截了本地请求。`llamapi-cli` 已内置本地代理绕过逻辑，仍有问题时执行代理屏蔽命令：

```bash
export no_proxy=localhost,127.0.0.1,::1
```

### 健康检查连接失败

依次检查：

1. `systemctl status llamapi-server` 是否显示服务运行。
2. `/etc/llamapi-server/config.toml` 中端口是否为 `9265`。
3. `journalctl -u llamapi-server -b` 是否有监听失败或配置错误。
4. 端口是否被其他进程占用。

本文档中的 `llamapi-cli`、配置和 API 示例统一使用端口 `9265`。

### `llamapi-server` 启动失败

查看本次启动日志：

```bash
journalctl -u llamapi-server -b
```

如果日志包含 `failed to parse config file`，检查 TOML：

- 表名和字段名是否正确。
- 字符串是否使用引号闭合。
- `[[models]]` 是否使用双层方括号。
- `instance_count` 是否大于等于 `1`。
- `model_path` 是否存在。

修正后重启：

```bash
sudo systemctl restart llamapi-server
```

### 未检测到可用的推理平台

执行：

```bash
llamapi platform
curl -s http://127.0.0.1:9265/v1/platforms
```

未检测到可用的推理平台通常表示：

- 当前没有检测到对应芯片。
- 后端运行库未安装或无法加载。
- Debian 包缺少所需动态库。
- 模型平台与设备硬件不匹配。

`llamapi-cli` 只会使用 `/v1/platforms` 中 `available: true` 且芯片匹配的平台。

### 指定模型使用的推理平台失败

显式指定平台和芯片：

```bash
llamapi pull qwen3:4b --platform rknn3/rk1828
llamapi run qwen3:4b --platform rknn3/rk1828
```

如果 `/v1/platforms` 没有将该组合标记为可用，`llamapi-cli` 会拒绝运行。

## 模型查询、下载与选择

### 下载模型返回 401 或 404

可能是远程仓库不存在或模型名不匹配。

1. 获取当前硬件可直接使用的模型名称：

   ```bash
   llamapi list --online
   ```

2. 从列表中复制正确的模型名称，例如使用 `bge-m3`，而不是 `bge_m3`。
3. 切换下载源：

   ```bash
   llamapi pull qwen3:4b --source modelscope
   llamapi pull qwen3:4b --source huggingface
   ```

### 远程模型列表查询超时

默认 `auto` 模式会并发访问 Hugging Face 和 ModelScope。全部来源都失败时，错误信息会列出每个来源的原因和尝试次数。

可以：

- 检查设备网络和 DNS。
- 明确指定可访问的下载源。
- 在 `llamapi-cli` 配置中将 `download.source` 设置为 `modelscope` 或 `huggingface`。
- 使用 `llamapi list --all` 判断问题是否与 `llamapi-server` 平台检测有关。

### 同名模型存在多个平台变体

交互终端会提示选择。脚本、管道和其他非交互环境中，应明确指定：

```bash
llamapi pull qwen3:4b --platform rkllm/rk3588
llamapi load qwen3:4b --platform rkllm/rk3588
```

`run` 可以在已维护性能排名的对话模型中自动选择，但显式 `--platform` 始终优先。

## 模型加载与实例管理

### `llamapi run` 提示模型为 Embedding 类型

Embedding 模型不支持对话。先加载模型：

```bash
llamapi load bge-m3
llamapi ps
```

然后调用：

```bash
curl http://127.0.0.1:9265/v1/embeddings \
  -H "Content-Type: application/json" \
  -d '{
    "model": "bge-m3@rknn2-rk3588",
    "input": "需要转换为向量的文本"
  }'
```

实际模型 ID 以 `llamapi ps` 命令显示的信息为准。


### 模型预加载失败但服务仍然运行

正常。单个 `[[models]]` 条目加载失败不会阻止 HTTP 服务启动。

检查：

```bash
journalctl -u llamapi-server -b
curl -s http://127.0.0.1:9265/v1/models
```

重点确认：

- `model_path` 指向正确目录。
- 模型目录包含有效 `model.toml`。
- 模型文件完整。
- 平台后端在当前硬件上可用。

### API 返回 `queue_full`

这表示所有实例都在处理请求，而且等待队列已满。

可以：

- 增加实例数：

  ```bash
  llamapi load <runtime-id> --instance 2
  ```

- 增加 `server.request_queue_size` 或模型级 `request_queue_size`。
- 限制客户端并发和重试频率。

增加实例数会占用更多硬件资源，并且实际成功数量可能小于请求数量。

### 请求的模型实例仅部分加载

`llamapi-server` 允许部分成功。响应或 `llamapi-cli` 输出会同时给出目标数量和实际数量，例如：

```text
model-id partially loaded: 2/4 instances active
```

这通常表示硬件资源不足或部分实例初始化失败。检查服务日志，并以 `llamapi ps` 或 `/v1/models` 中的实际实例数为准。

### 协处理器加载多个模型实例后通信失败

在协处理器芯片上加载多个模型实例可能出现以下故障：

- 模型实例加载失败。
- 协处理器芯片通信失败。
- `rknn3.service` 或 `llamapi-server.service` 状态异常。
- 后续平台查询、模型加载或推理请求失败。

发生故障后，停止继续加载模型或发送推理请求，并严格按照以下顺序执行恢复命令：

```bash
sudo rknn-smi reset
sudo rknn-smi reset -t hw
systemctl restart rknn3.service
systemctl restart llamapi-server.service
```

执行顺序不能调换：先执行两次芯片重置，再重启 `rknn3.service`，最后重启 `llamapi-server.service`。如果当前用户没有管理 systemd 服务的权限，请为最后两条命令添加 `sudo`。

恢复后检查服务和芯片状态：

```bash
systemctl status rknn3.service
systemctl status llamapi-server.service
llamapi platform
curl -s http://127.0.0.1:9265/health
```

为避免再次触发该问题，在协处理器芯片上加载模型时，当前建议将 `instance_count` 或 `--instance` 保持为 `1`。

### `run` 异常退出后模型未卸载

正常退出、`Ctrl+C`、SIGTERM 和大多数执行错误会尝试卸载本次加载的模型。SIGKILL、进程崩溃和断电无法执行清理。

检查并手动卸载：

```bash
llamapi ps
llamapi unload <runtime-id>
```

## HTTP API 请求

### API 返回 `model_not_found`

请求中的 `model` 必须与已加载模型 ID 完全一致。

```bash
llamapi ps
curl -s http://127.0.0.1:9265/v1/models
```

不要使用本地显示名称代替运行时 ID，除非两者恰好相同。

### API 返回 `wrong_model_type`

- `/v1/chat/completions` 必须使用 `model_kind=chat` 的模型。
- `/v1/embeddings` 必须使用 `model_kind=embedding` 的模型。

通过 `/v1/models` 检查 `model_kind`。

### API 返回 `context_length_exceeded`

请求内容和预期输出超过模型上下文上限。

- 减少历史消息或输入内容。
- 减小 `max_tokens` 或 `max_completion_tokens`。
- 检查模型的 `max_context_len` 配置。

### 远程图片 URL 不可用

`image_url` 当前只支持：

- `llamapi-server` 本地文件路径。
- `png`、`jpeg`、`jpg`、`webp` 的 base64 data URL。

不支持 `http://` 或 `https://` 图片 URL。客户端需要先下载图片，或将图片编码为 data URL。

## 诊断信息

### 收集诊断信息

建议收集以下信息：

```bash
llamapi --version
llamapi platform
llamapi list
llamapi ps
systemctl status llamapi-server
journalctl -u llamapi-server -b
curl -s http://127.0.0.1:9265/v1/platforms
curl -s http://127.0.0.1:9265/v1/models
```

同时记录设备型号、芯片型号、执行命令、完整错误输出和 `llamapi-server` 配置中相关字段。提交日志前应检查是否包含敏感路径或业务数据。
