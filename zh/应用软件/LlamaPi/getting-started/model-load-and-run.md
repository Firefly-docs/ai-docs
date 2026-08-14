# 模型加载与运行

本章介绍如何运行对话模型、加载模型以持续提供服务，以及查看和卸载已加载模型。

## 运行对话模型

进入交互式对话：

```bash
llamapi run qwen3:4b
```

执行单次提问：

```bash
llamapi run qwen3:4b "请解释什么是 NPU"
```

设置系统提示词：

```bash
llamapi run qwen3:4b --system "你是一名嵌入式 Linux 工程师"
```

`run` 会按需下载和加载模型，退出时会卸载由本次命令加载的模型。它适合临时对话和单次提问。

## 使用交互模式

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

## 加载模型

如果模型需要在多个请求之间保持可用，使用：

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

## 查看已加载模型

```bash
llamapi ps
```

该命令显示模型的运行时 ID、类型、推理平台、运行状态、实例数和自动加载状态。

## 调整实例数

将已加载的模型调整为 3 个实例：

```bash
llamapi load assistant --instance 3
```

实例数会增加模型的并行处理能力，也会占用更多硬件资源。请根据设备资源进行设置。

## 卸载模型

```bash
llamapi unload assistant
```

卸载模型不会删除本地模型文件。

## 使用 Embedding 模型

Embedding 模型不能通过 `llamapi run` 进行对话。先下载并加载模型：

```bash
llamapi pull bge-m3
llamapi load bge-m3
llamapi ps
```

然后调用 `/v1/embeddings`。具体请求方式见[API 接口详解](../advanced-guides/api-reference.md#embeddings)。

如需让模型在 LlamaPi 服务启动时自动加载，请继续阅读[持久化部署](./model-persistence.md)。
