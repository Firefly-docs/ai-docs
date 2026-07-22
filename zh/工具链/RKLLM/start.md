# RKLLM 简介

RKLLM SDK 帮助开发者将 Hugging Face 格式的大语言模型部署到 Rockchip NPU 平台。完整工具链包含 PC 端的 RKLLM-Toolkit 和板端的 RKLLM Runtime。

## RKLLM-Toolkit

RKLLM-Toolkit 在 Linux PC 上运行，通过 Python API 提供：

- **模型转换**：将 Hugging Face 格式的 LLM 转换为 RKLLM 模型。
- **模型量化**：将浮点模型量化为定点模型，支持的量化类型以当前 SDK 文档为准，常用类型包括 W4A16 和 W8A8。

## RKLLM Runtime

RKLLM Runtime 加载 RKLLM-Toolkit 导出的模型，调用板端 NPU 驱动完成推理。应用程序可以设置生成参数，并通过回调函数持续获取生成结果。

```text
Hugging Face 模型
        ↓
RKLLM-Toolkit（Linux PC）
        ├── 模型转换
        └── 模型量化
                 ↓
             .rkllm 模型
                 ↓
RKLLM Runtime（Rockchip 板端）
                 ↓
              NPU 推理
```
