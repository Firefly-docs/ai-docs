# RKLLM-Toolkit

RKLLM-Toolkit currently runs on a Linux x86_64 PC. Ubuntu 20.04 and a Conda-managed Python environment are recommended.

## Installation

```bash
conda create -n RKLLM-Toolkit python=3.8
conda activate RKLLM-Toolkit

pip3 install rkllm-toolkit/rkllm_toolkit-1.1.4-cp38-cp38-linux_x86_64.whl
```

The wheel name above is only an example. Install the file included with the SDK that matches the selected Python version.

Verify the installation:

```python
from rkllm.api import RKLLM
```

## Supported Models

RKLLM supports multiple mainstream LLMs and VLMs. The exact support matrix changes between SDK releases. Common model families include:

| Model | Reference |
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
