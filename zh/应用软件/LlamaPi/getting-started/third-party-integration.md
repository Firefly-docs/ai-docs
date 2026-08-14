# 接入第三方应用

LlamaPi 可以通过 OpenAI 兼容 API 为第三方应用提供本地大模型能力。只要应用支持自定义 OpenAI Base URL，通常就可以填写 LlamaPi 的服务地址、模型 ID 和 API Key 后进行连接。

本章以对话模型为例介绍通用接入方法。接口字段和完整能力范围见[API 接口详解](../advanced-guides/api-reference.md)。

## 接入前提

配置第三方应用前，先确认服务、模型和网络三项状态。服务和模型检查应在运行 LlamaPi 的设备上执行，网络检查应在第三方应用所在设备上执行。

### 确认服务和推理平台

```bash
llamapi platform
```

正常输出示例：

```text
SoC:                    RK3588
Coprocessor:            null
Supported platforms:    rkllm, rknn2
Unsupported platforms:  rknn3
Service:                running  (http://127.0.0.1:9265/v1)
```

再检查健康接口：

```bash
curl -s http://127.0.0.1:9265/health
```

正常情况下返回：

```text
ok
```

判断方法：

- `Service: running` 表示 LlamaPi 服务正在运行，并且 API 服务地址为 `http://127.0.0.1:9265/v1`。
- `/health` 返回单独一行 `ok`，表示 HTTP 服务可以正常响应。
- 如果显示 `Service: not running`、`/health` 无法连接，或 `Supported platforms` 为 `none`，则暂时不满足接入前提。请先启动服务或排查推理平台状态。

### 确认模型已加载

```bash
llamapi ps
```

正常输出示例：

```text
ID                         MODEL       TYPE   PLATFORM       STATUS       INSTANCES    ENABLE
qwen3:4b@rkllm-rk3588      qwen3:4b    chat   rkllm/rk3588   ● active     1            no
```

判断方法：

- 至少有一行模型记录，且 `TYPE` 为 `chat`、`STATUS` 为 `● active`，才可以接收对话请求。
- 将 `ID` 列的完整值作为第三方应用中的模型名称。不要只填写 `MODEL` 列中的显示名称。
- 如果没有记录，或状态为 `○ inactive`，先加载模型：

```bash
llamapi load qwen3:4b
```

### 确认网络可达

在第三方应用所在设备上，将下面地址中的 IP 替换为运行 LlamaPi 的设备实际 IP：

```bash
curl -s http://192.168.1.100:9265/health
```

返回 `ok` 表示第三方应用所在设备可以访问 LlamaPi 服务；如果连接超时或被拒绝，则还不满足接入前提，需要检查设备 IP、防火墙和端口 `9265`。

## 准备连接信息

第三方应用通常需要以下配置：

| 配置项 | 填写内容 |
|:---:|:---:|
| OpenAI Base URL | `http://<设备 IP>:9265/v1` |
| 模型 | `llamapi ps` 显示的模型 ID |
| API Key | 如果应用要求必填，可填写任意非空内容 |

上面的检查结果中，模型 ID 为：

```text
qwen3:4b@rkllm-rk3588
```

## 选择服务地址

如果第三方应用与 LlamaPi 运行在同一设备上，使用：

```text
http://127.0.0.1:9265/v1
```

如果第三方应用位于同一局域网中的其他设备上，将 `<设备 IP>` 替换为运行 LlamaPi 的 Firefly 设备 IP：

```text
http://192.168.1.100:9265/v1
```

可以在运行 LlamaPi 的设备上使用以下命令查看 IP 地址：

```bash
hostname -I
```

## 使用 curl 验证接口

建议先在能够访问 LlamaPi 软件服务的终端中验证对话接口：

```bash
curl http://127.0.0.1:9265/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "qwen3:4b@rkllm-rk3588",
    "messages": [
      {"role": "user", "content": "你好"}
    ],
    "stream": false
  }'
```

将 `model` 替换为 `llamapi ps` 显示的实际模型 ID。通过局域网访问时，还需要将 `127.0.0.1` 替换为运行 LlamaPi 的设备 IP。

## 配置第三方应用

不同应用的设置名称可能不同，但通用配置流程如下：

1. 在应用中选择 OpenAI 或 OpenAI-compatible 服务提供方。
2. 找到自定义 Base URL、API Endpoint 或服务地址的设置。
3. 填写 `http://<设备 IP>:9265/v1`。
4. 将模型名称填写为 `llamapi ps` 显示的模型 ID。
5. 如果 API Key 为必填项，填写任意非空内容。
6. 保存配置并发送一条测试消息。

如果应用会自动查询模型列表，应确认它访问的是 `/v1/models`。如果应用不能自定义 Base URL，或者只接受特定云服务地址，则无法使用这种通用方式直接接入。

> ⚠️ <strong style="color: #dc2626;">安全提醒</strong>：当前 LlamaPi 不提供 API 鉴权。第三方应用中填写的 API Key 仅用于满足客户端的必填校验，不会被 LlamaPi 验证，也不能限制访问。建议仅在本机或可信局域网中使用，不要直接将端口 `9265` 暴露到公网。需要跨网络访问时，请配置防火墙、反向代理和访问认证。详细说明见[服务配置与运维](../advanced-guides/server-operations.md#配置安全与网络访问)。

## 常见连接问题

| 现象 | 检查方法 |
|:---:|:---:|
| 无法连接服务 | 检查设备 IP、端口、LlamaPi 服务状态和网络连通性 |
| 找不到模型 | 执行 `llamapi ps`，确认模型已加载并使用完整模型 ID |
| API Key 报空 | 在第三方应用中填写任意非空值 |
| 对话接口返回模型类型错误 | 确认使用的是 `chat` 模型，而不是 Embedding 模型 |
| 局域网可达但请求失败 | 检查设备防火墙是否允许访问端口 `9265` |

更多问题见[常见问题与故障排查](../llamapi/faq.md)。
