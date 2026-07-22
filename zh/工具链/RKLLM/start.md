# 1.RKLLM 介绍
RKLLM SDK可以帮助用户快速将大语言模型部署到RK3588 等 Rockchip NPU 平台上。
[SDK下载](https://github.com/airockchip/rknn-llm)
#### 1.1 RKLLM-Toolkit 功能介绍
RKLLM-Toolkit 是为用户提供在计算机上进行大语言模型的量化、转换的开发套件。通过该工具提供的 Python 接口可以便捷地完成以下功能：
* 模型转换：支持将 Hugging Face 格式的大语言模型（Large Language Model, LLM）转换为RKLLM 模型，转换后的 RKLLM 模型能够在 Rockchip NPU 平台上加载使用。
* 量化功能：支持将浮点模型量化为定点模型，目前支持的量化类型包括 w4a16 和 w8a8。
#### 1.2 RKLLM Runtime 功能介绍
RKLLM Runtime 主 要 负 责 加 载 RKLLM-Toolkit 转换得到的 RKLLM 模型，并在RK3588 等主控平台通过调用 NPU 驱动在 Rockchip NPU 上实现 RKLLM 模型的推理。在推理RKLLM 模型时，用户可以自行定义 RKLLM 模型的推理参数设置，定义不同的文本生成方式，并通过预先定义的回调函数不断获得模型的推理结果。
