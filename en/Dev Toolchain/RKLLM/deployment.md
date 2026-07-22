# 3.Large Language Model Deployment Example

The large language models supported by RKLLM are as follows:

| Model               | Huggingface Link                                                          					|
| ------------------- | ------------------------------------------------------------------------------------------------------ |
| DeepSeek-R1-Distill | [LINK](https://huggingface.co/collections/deepseek-ai/deepseek-r1-678e1e131c0169c0bc89728d)            |
| LLAMA               | [LINK](https://huggingface.co/meta-llama)                                                              |
| TinyLLAMA           | [LINK](https://huggingface.co/TinyLlama)                                                               |
| Qwen                | [LINK](https://huggingface.co/models?search=Qwen/Qwen)                                                 |
| Phi                 | [LINK](https://huggingface.co/models?search=microsoft/phi)                                             |
| ChatGLM3-6B         | [LINK](https://huggingface.co/THUDM/chatglm3-6b/tree/103caa40027ebfd8450289ca2f278eac4ff26405)         |
| Gemma               | [LINK](https://huggingface.co/collections/google/gemma-2-release-667d6600fd5220e7b967f315)             |
| InternLM2           | [LINK](https://huggingface.co/collections/internlm/internlm2-65b0ce04970888799707893c)                 |
| MiniCPM             | [LINK](https://huggingface.co/collections/openbmb/minicpm-65d48bf958302b9fd25b698f)                    |
| TeleChat            | [LINK](https://huggingface.co/Tele-AI)                                                                 |
| Qwen2-VL            | [LINK](https://huggingface.co/Qwen/Qwen2-VL-2B-Instruct)                                               |
| MiniCPM-V           | [LINK](https://huggingface.co/openbmb/MiniCPM-V-2_6)                                                   |

#### 3.1 Large Language Model Conversion And Execution Demo
Below is an example demonstrating how to convert, quantize, export a large language model, and finally deploy and run it on the board, using the `DeepSeek-R1-Distill-Qwen-1.5B_Demo` provided by the RKLLM SDK.
##### 3.1.1 Convert The Model On The PC
Using DeepSeek-R1-Distill-Qwen-1.5B as an example, click the table link and find the corresponding repository to clone the [complete repository](https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-1.5B/tree/main).

With RKLLM-Toolkit properly installed, activate the Conda base environment and execute `generate_data_quant.py`. This will generation results as the quantization calibration data.
```
# Note that you need to set the path to the repository that you cloned.
cd export
python3 generate_data_quant.py -m ~/DeepSeek-R1-Distill-Qwen-1.5B
```
Modify `export_rkllm.py` according to the actual situation. Execute `export_rkllm.py`.You can get the rkllm model.
```
modelpath = '/rkllm/rkllm_model/DeepSeek-R1-Distill-Qwen-1.5B'
# For RK3576, Quantization type is W4A16,and it is a dual-core NPU.Then no change is required for RK3588.
target_platform = "RK3576"
quantized_dtype = "W4A16"
num_npu_core = 2
```

```
(RKLLM-Toolkit-1.1.4) root@ea5d57ca8e66:/rkllm/rknn-llm/examples/DeepSeek-R1-Distill-Qwen-1.5B_Demo/export# python3 export_rkllm.py 
INFO: rkllm-toolkit version: 1.1.4
WARNING: Cuda device not available! switch to cpu device!
The argument `trust_remote_code` is to be used with Auto classes. It has no effect here and is ignored.
Downloading data files: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1/1 [00:00<00:00, 15307.68it/s]
Extracting data files: 100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1/1 [00:00<00:00, 2377.72it/s]
Generating train split: 21 examples [00:00, 3327.05 examples/s]
Optimizing model: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 28/28 [21:16<00:00, 45.59s/it]
Building model: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 399/399 [00:07<00:00, 52.78it/s]
WARNING: The bos token has two ids: 151646 and 151643, please ensure that the bos token ids in config.json and tokenizer_config.json are consistent!
INFO: The token_id of bos is set to 151646
INFO: The token_id of eos is set to 151643
INFO: The token_id of pad is set to 151643
Converting model: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 339/339 [00:00<00:00, 2494507.12it/s]
INFO: Exporting the model, please wait ....
[=================================================>] 597/597 (100%)
INFO: Model has been saved to ./DeepSeek-R1-Distill-Qwen-1.5B_W8A8_RK3588.rkllm!
```
##### 3.1.2 Deploy And Run The RKLLM Model On a Target Platform Such as RK3588.

###### 3.1.2.1 Kernel Requirements
Before performing model inference with RKLLM Runtime, you must first verify that the NPU kernel on the board is version `v0.9.8` or higher.
```
root@firefly:/# cat /sys/kernel/debug/rknpu/version
RKNPU driver: v0.9.8
```
If the NPU kernel version is lower than v0.9.8, please go to the official firmware address to download the latest firmware
##### 3.1.2.2 Compilation Requirements for RKLLM Runtime
When using RKLLM Runtime, pay attention to the version of the GCC cross-compilation toolchain. It is recommended to use cross-compilation toolchain [gcc-arm-10.2-2020.11-x86_64-aarch64-none-linux-gnu](https://developer.arm.com/downloads/-/gnu-a/10-2-2020-11)
```
cd deploy
# Modify build-linux.sh
GCC_COMPILER_PATH=~/gcc-arm-10.2-2020.11-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu
# Add executable permissions
chmod +x ./build-linux.sh
# Compilation
./build-linux.sh
```
###### 3.1.2.3 Running Inference On a Target Platform Such as RK3588
After compiling, generate the executable program `install/demo_Linux_aarch64/llm_demo`. Transfer the compiled `llm_demo` executable and the library file `rkllm-runtime/runtime/Linux/librkllm_api/aarch64/librkllmrt.so` and fixed frequency script `rknn-llm/scripts/fix_freq_rk3588.sh` to the board using `scp` command.
```
firefly@firefly:~$ ./fix_freq_rk3588.sh
firefly@firefly:~$ export LD_LIBRARY_PATH=./lib
firefly@firefly:~$ taskset f0 ./llm_demo ./DeepSeek-R1-Distill-Qwen-1.5B_W8A8_RK3588.rkllm 2048 4096
rkllm init start
I rkllm: rkllm-runtime version: 1.1.4, rknpu driver version: 0.9.8, platform: RK3588

rkllm init success

**********************可输入以下问题对应序号获取回答/或自定义输入********************

[0] 现有一笼子，里面有鸡和兔子若干只，数一数，共有头14个，腿38条，求鸡和兔子各有多少只？
[1] 有28位小朋友排成一行,从左边开始数第10位是学豆,从右边开始数他是第几位?

*************************************************************************


user: who are you?
robot: <think>

</think>

Greetings! I'm DeepSeek-R1, an artificial intelligence assistant created by DeepSeek. I'm at your service and would be delighted to assist you with any inquiries or tasks you may have.

user: 
```
### 4.Others
For more demo and api usage, refer to RKLLM SDK routines and documentation
