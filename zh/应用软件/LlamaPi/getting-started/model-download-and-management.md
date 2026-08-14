# 模型下载与管理

本章介绍如何查找当前硬件可用的模型，并下载、查看和删除本地模型。

## 模型名称与类型

LlamaPi 支持带参数规模和不带参数规模的模型名称：

| 格式 | 示例 |
|:---:|:---:|
| `<model>:<size>` | `qwen3:4b` |
| `<model>` | `bge-m3` |

当前模型分为两类：

| 类型 | 用途 |
|:---:|:---:|
| `chat` | 对话、文本生成和工具调用 |
| `embedding` | 将文本转换为向量 |

同一个模型可能包含适用于不同推理平台和硬件的变体。通常由 LlamaPi 根据当前硬件自动选择；需要手动指定时，使用 `--platform <platform>/<chip>`。

## 当前支持的推理平台

| 推理平台 | 模型类型 | 当前支持的硬件 |
|:---:|:---:|:---:|
| `rknn3` | `chat` | RK1820、RK1828 协处理器或 RK3572 SoC |
| `rkllm` | `chat` | RK3576、RK3588、RK3562、RV1126B 等 SoC |
| `rknn2` | `embedding` | 支持 RKNN2 的 NPU 平台 |

LlamaPi 会检测当前硬件和可用推理平台。后续版本会继续扩展硬件和推理平台支持。

## 查找当前硬件可用的模型

```bash
llamapi list --online
```

该命令只显示适用于当前硬件的远程模型。模型列表默认来自 Hugging Face 或 ModelScope。

如需查看远程仓库中的全部模型，不按当前硬件筛选：

```bash
llamapi list --all
```

## 下载模型

```bash
llamapi pull qwen3:4b
```

指定推理平台和硬件：

```bash
llamapi pull qwen3:4b --platform rkllm/rk3588
```

指定下载源：

```bash
llamapi pull qwen3:4b --source modelscope
```

下载完成后，LlamaPi 会验证模型文件并显示模型大小。下载模型不会自动加载模型。

## 查看本地模型

```bash
llamapi list
```

本地模型默认保存在：

```text
/var/lib/llamapi/models
```

## 删除本地模型

```bash
llamapi rm qwen3:4b
```

删除指定推理平台的模型变体：

```bash
llamapi rm qwen3:4b --platform rkllm/rk3588
```

如果模型正在运行，LlamaPi 会询问是否先卸载模型。删除本地文件后，再次使用该模型时需要重新下载。

下一步可以阅读[模型加载与运行](./model-load-and-run.md)。完整命令参数见[终端命令详解](../advanced-guides/cli-command-guide.md)。
