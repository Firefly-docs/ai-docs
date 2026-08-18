# 模型运行与部署

本章介绍如何使用 `run` 运行模型、使用 `load` 部署模型。

[下载模型](./model-download-and-management.md)只是将模型文件保存到本地。开始推理前，LlamaPi 还需要将模型加载到对应的推理平台，并创建可接收请求的运行时模型。已加载的模型会占用设备的内存资源。

`run` 和 `load` 都会加载模型，但用途和生命周期不同：

| 使用方式 | 执行行为 | 适用场景 |
|:---:|:---:|:---:|
| `llamapi run` | 按需启动服务、下载并加载模型；退出时卸载本次命令新加载的模型 | 交互式对话、单次提问 |
| `llamapi load` | 加载已下载的模型；命令结束后模型仍保持部署，直到主动卸载或服务停止 | 持续提供 API 服务、接入第三方应用 |

如果 `run` 复用了已通过 `load` 部署的模型，退出 `run` 不会卸载该模型。

## 运行模型（`run`）

`run` 会自动完成模型查找、按需下载和加载，然后进入交互式对话或执行单次提问。

进入交互式对话：

```bash
llamapi run qwen3:4b
```

执行单次提问：

```bash
llamapi run qwen3:4b "请解释什么是 NPU"
```

设置系统提示词并进入交互式对话：

```bash
llamapi run qwen3:4b --system "你是一名嵌入式 Linux 工程师"
```

退出 `run` 后，由本次命令新加载的模型将被自动卸载，不再继续接收 API 请求。如果需要模型持续可用，请使用 `load` 进行部署。

### 使用交互模式

进入交互模式后，可以使用：

| 命令 | 说明 |
|:---:|:---:|
| `/help` | 显示可用命令 |
| `/clear` | 清空对话历史 |
| `/system <prompt>` | 设置或清除系统提示词 |
| `/set <key> <value>` | 修改生成参数 |
| `/show info` | 显示当前模型信息 |
| `/show params` | 显示当前生成参数 |
| `/exit` | 退出对话 |

输入 `"""` 可以开始或结束多行输入。生成过程中按 `Ctrl+C` 可中断当前回答。

## 部署模型（`load`）

`load` 会将已下载的模型加载到 LlamaPi 服务。加载完成后命令即可结束，关闭当前终端不会影响已部署的模型。

使用 `load` 前，需要先下载模型并确保 LlamaPi 服务正在运行。如果服务未运行，可以执行：

```bash
sudo systemctl start llamapi-server
```

加载并部署一个模型实例：

```bash
llamapi load qwen3:4b
```

加载多个实例：

```bash
llamapi load qwen3:4b --instance 2
```

指定运行时 ID：

```bash
llamapi load qwen3:4b --id assistant
```

第三方应用和 API 请求应使用 `llamapi ps` 显示的实际运行时 ID。

### 查看已部署模型

```bash
llamapi ps
```

该命令显示模型的运行时 ID、类型、推理平台、运行状态、实例数和自动加载状态。

### 调整实例数

将已部署的模型调整为 3 个实例：

```bash
llamapi load assistant --instance 3
```

实例数会增加模型的并行处理能力，也会占用更多硬件资源。请根据设备资源进行设置。

### 卸载模型

```bash
llamapi unload assistant
```

卸载模型会结束该模型的部署并释放所占用的运行资源，但不会删除本地模型文件。

### 使用 Embedding 模型

Embedding 模型不能通过 `llamapi run` 进行对话。先下载并部署模型：

```bash
llamapi pull bge-m3
llamapi load bge-m3
llamapi ps
```

然后调用 `/v1/embeddings`。具体请求方式见[API 接口详解](../advanced-guides/api-reference.md#embeddings)。

`load` 的部署状态仅在当前 LlamaPi 服务运行期间有效。如需让模型在 LlamaPi 服务启动时自动加载，请继续阅读[持久化部署](./model-persistence.md)。
