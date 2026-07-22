# RKLLM Introduction

The RKLLM SDK helps developers deploy large language models in Hugging Face format on Rockchip NPU platforms. The complete toolchain consists of RKLLM-Toolkit on the PC and RKLLM Runtime on the target device.

## RKLLM-Toolkit

RKLLM-Toolkit runs on a Linux PC and provides Python APIs for:

- **Model conversion**: converts a Hugging Face LLM into an RKLLM model.
- **Model quantization**: quantizes floating-point models to fixed-point representations. Supported types depend on the SDK release and commonly include W4A16 and W8A8.

## RKLLM Runtime

RKLLM Runtime loads models exported by RKLLM-Toolkit and invokes the NPU driver to perform inference. Applications can configure generation parameters and continuously receive generated output through callback functions.

```text
Hugging Face model
        ↓
RKLLM-Toolkit (Linux PC)
        ├── Model conversion
        └── Model quantization
                 ↓
             .rkllm model
                 ↓
RKLLM Runtime (Rockchip target)
                 ↓
              NPU inference
```
