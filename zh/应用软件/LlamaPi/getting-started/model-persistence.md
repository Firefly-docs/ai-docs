# 持久化部署

本章介绍如何让模型在设备启动时自动加载，从而确保 LlamaPi 持续为外部应用提供服务。

## 设置自动加载

```bash
llamapi enable qwen3:4b
```

该命令会保存模型的自动加载设置，设备下次启动时会自动加载模型。

## 立即加载并验证

设置自动加载并立即加载模型：

```bash
llamapi enable qwen3:4b --now
```

查看模型状态：

```bash
llamapi ps
```

`ENABLE` 列显示 `yes(N)` 时，表示该模型已配置为自动加载，`N` 为配置的实例数。

## 配置实例数和运行时 ID

```bash
llamapi enable qwen3:4b --instance 2 --id assistant --now
```

实例数会影响硬件资源占用。请根据设备资源进行设置。

## 取消自动加载

```bash
llamapi disable qwen3:4b
```

取消自动加载并立即卸载模型：

```bash
llamapi disable qwen3:4b --now
```

如果配置时使用了自定义运行时 ID，应使用该 ID 取消配置：

```bash
llamapi disable assistant --now
```

如需进一步配置和排查 LlamaPi 服务，请阅读[服务配置与运维](../advanced-guides/server-operations.md)。完整命令参数见[终端命令详解](../advanced-guides/cli-command-guide.md)。
