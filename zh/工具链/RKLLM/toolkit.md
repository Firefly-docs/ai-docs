# RKLLM-Toolkit

RKLLM-Toolkit 目前运行在 Linux x86_64 PC 上，建议使用 Ubuntu 20.04 并通过 Conda 管理 Python 环境。

## 安装

```bash
conda create -n RKLLM-Toolkit python=3.8
conda activate RKLLM-Toolkit

pip3 install rkllm-toolkit/rkllm_toolkit-1.1.4-cp38-cp38-linux_x86_64.whl
```

上述 wheel 文件名仅为示例。实际安装时，应选择 SDK 中与 Python 版本匹配的文件。

验证安装：

```python
from rkllm.api import RKLLM
```

## 支持的模型

RKLLM SDK 支持多种主流 LLM 和 VLM，具体支持范围随 SDK 版本变化。常见模型包括：

| 模型 | 参考地址 |
| --- | --- |
| DeepSeek-R1-Distill | [Hugging Face](https://huggingface.co/collections/deepseek-ai/deepseek-r1-678e1e131c0169c0bc89728d) |
| Llama | [Hugging Face](https://huggingface.co/meta-llama) |
| TinyLlama | [Hugging Face](https://huggingface.co/TinyLlama) |
| Qwen | [Hugging Face](https://huggingface.co/models?search=Qwen/Qwen) |
| Phi | [Hugging Face](https://huggingface.co/models?search=microsoft/phi) |
| ChatGLM3-6B | [Hugging Face](https://huggingface.co/THUDM/chatglm3-6b) |
| Gemma | [Hugging Face](https://huggingface.co/collections/google/gemma-2-release-667d6600fd5220e7b967f315) |
| InternLM2 | [Hugging Face](https://huggingface.co/collections/internlm/internlm2-65b0ce04970888799707893c) |
| MiniCPM | [Hugging Face](https://huggingface.co/collections/openbmb/minicpm-65d48bf958302b9fd25b698f) |
| Qwen2-VL | [Hugging Face](https://huggingface.co/Qwen/Qwen2-VL-2B-Instruct) |
| MiniCPM-V | [Hugging Face](https://huggingface.co/openbmb/MiniCPM-V-2_6) |
