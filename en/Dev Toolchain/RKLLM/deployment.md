# Model Conversion and On-device Deployment

This workflow uses `DeepSeek-R1-Distill-Qwen-1.5B_Demo` from the RKLLM SDK to demonstrate quantization, conversion, and target deployment.

## Convert the Model on a PC

Download the complete [DeepSeek-R1-Distill-Qwen-1.5B](https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-1.5B/tree/main) repository. In the Conda environment containing RKLLM-Toolkit, generate the quantization calibration data:

```bash
cd export
python3 generate_data_quant.py -m ~/DeepSeek-R1-Distill-Qwen-1.5B
```

Set the model path, target platform, and quantization type in `export_rkllm.py`. For example, for RK3576:

```python
modelpath = "/rkllm/rkllm_model/DeepSeek-R1-Distill-Qwen-1.5B"
target_platform = "RK3576"
quantized_dtype = "W4A16"
num_npu_core = 2
```

RK3588 and RK3576 may require different quantization and NPU core settings. Follow the examples included with the current SDK. Run the conversion:

```bash
python3 export_rkllm.py
```

## Target Environment

Before using RKLLM Runtime, check the NPU driver version:

```bash
cat /sys/kernel/debug/rknpu/version
```

The original AIBOX-PRO example requires RKNPU driver `v0.9.8` or later. Always follow the requirements of the current RKLLM SDK release.

## Build the Runtime Example

Set the aarch64 cross-compiler path and build the example:

```bash
cd deploy
GCC_COMPILER_PATH=~/gcc-arm-10.2-2020.11-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu
chmod +x ./build-linux.sh
./build-linux.sh
```

See the [GNU Arm Toolchain 10.2](https://developer.arm.com/downloads/-/gnu-a/10-2-2020-11) reference.

## Run on the Target Device

Deploy the following files to the target:

- `install/demo_Linux_aarch64/llm_demo`
- `rkllm-runtime/Linux/librkllm_api/aarch64/librkllmrt.so`
- The converted `.rkllm` model
- The platform-specific frequency script, if required

Run the example:

```bash
export LD_LIBRARY_PATH=./lib
taskset f0 ./llm_demo ./DeepSeek-R1-Distill-Qwen-1.5B_W8A8_RK3588.rkllm 2048 4096
```

## Chat Templates

Chat models use different prompt formats. The chat template must match the message format used during training, or output quality and performance may degrade. Prefer the `tokenizer_config.json` or official chat template provided by the model repository.
