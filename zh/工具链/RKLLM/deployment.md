# 模型转换与板端部署

以 RKLLM SDK 中的 `DeepSeek-R1-Distill-Qwen-1.5B_Demo` 为例，说明量化、转换和板端运行流程。

## PC 端转换

下载完整的 [DeepSeek-R1-Distill-Qwen-1.5B](https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-1.5B/tree/main) 模型仓库，然后在已安装 RKLLM-Toolkit 的 Conda 环境中生成量化标定数据：

```bash
cd export
python3 generate_data_quant.py -m ~/DeepSeek-R1-Distill-Qwen-1.5B
```

在 `export_rkllm.py` 中设置模型路径、目标平台和量化类型。例如 RK3576：

```python
modelpath = "/rkllm/rkllm_model/DeepSeek-R1-Distill-Qwen-1.5B"
target_platform = "RK3576"
quantized_dtype = "W4A16"
num_npu_core = 2
```

RK3588 与 RK3576 的量化类型和 NPU 核配置可能不同，请以当前 SDK 示例为准。执行转换：

```bash
python3 export_rkllm.py
```

## 板端环境

使用 RKLLM Runtime 前，先确认 NPU 驱动版本：

```bash
cat /sys/kernel/debug/rknpu/version
```

原 AIBOX-PRO 示例要求 RKNPU 驱动不低于 `v0.9.8`。实际要求应以当前 RKLLM SDK 发布说明为准。

## 编译 Runtime 示例

设置 aarch64 交叉编译器路径并编译：

```bash
cd deploy
GCC_COMPILER_PATH=~/gcc-arm-10.2-2020.11-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu
chmod +x ./build-linux.sh
./build-linux.sh
```

可参考 [GNU Arm Toolchain 10.2](https://developer.arm.com/downloads/-/gnu-a/10-2-2020-11)。

## 板端运行

将以下文件部署到目标板：

- `install/demo_Linux_aarch64/llm_demo`
- `rkllm-runtime/Linux/librkllm_api/aarch64/librkllmrt.so`
- 已转换的 `.rkllm` 模型
- 对应平台的定频脚本（可选）

运行示例：

```bash
export LD_LIBRARY_PATH=./lib
taskset f0 ./llm_demo ./DeepSeek-R1-Distill-Qwen-1.5B_W8A8_RK3588.rkllm 2048 4096
```

## 聊天模板

不同聊天模型的 prompt 格式不同。聊天模板必须与模型训练时使用的消息格式匹配，否则可能导致输出质量下降。请优先使用模型仓库提供的 `tokenizer_config.json` 或官方聊天模板。
