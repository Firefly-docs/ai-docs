# 1.RKLLM Introduction
The RKLLM SDK helps users quickly deploy large language models on Rockchip NPU platforms such as RK3588.
[SDK Download](https://github.com/airockchip/rknn-llm)

#### 1.1 RKLLM-Toolkit Functions Introduction
The RKLLM-Toolkit is a development suite designed for users to perform quantization and conversion of large language models on their computers. Through the Python interface provided by this tool, users can conveniently achieve the following functions:
* Model Conversion: Supports converting large language models in Hugging Face format to RKLLM models. The converted RKLLM models can be loaded and used on the Rockchip NPU platform.
* Quantization: Supports quantizing floating-point models to fixed-point models. Currently supported quantization types include w4a16 and w8a8.

#### 1.2 RKLLM Runtime Functions Introduction
The RKLLM Runtime is primarily responsible for loading RKLLM models converted using the RKLLM-Toolkit and performing inference on the Rockchip NPU by invoking the NPU driver on target platforms such as RK3588. During the inference of RKLLM models, users can customize the inference parameters, define various text generation methods, and continuously receive inference results through predefined callback functions.
