# 1. Model Farm Guide

[Model Farm](https://aiot.aidlux.com/en/models) is the model marketplace for AidLux. It provides open-source models optimized for Qualcomm edge chipsets, runnable preprocessing, postprocessing, and inference examples, and performance figures measured on the stated hardware. Browsing models and performance data does not require an account; downloading model packages and example code does.

Model Farm supports Qualcomm QCS6490, QCS8550, QCS8625, Dragonwing IQ9, and Dragonwing IQ8 platforms. Downloaded models can run with [AidLux](../../Dev%20Toolchain/Aidlux/index.md) through AidLite, AidGen, or Qualcomm QNN.

## 1.1 Selecting a Model

Use model type, precision, chipset, and keyword filters to locate a model. A model detail page provides the following evaluation data:

| Field | Description |
| --- | --- |
| Device | Development board and chipset used for the measurement. |
| AI framework | Framework and version used for model conversion and inference. |
| Model precision | Such as FP16, INT8, or W4A16. |
| Inference latency | Measured inference time excluding preprocessing and postprocessing. |
| Accuracy loss | Cosine similarity comparison between the converted model output and the FP32 source model output. |
| Model size | Size of the converted model file. |

Performance on another board with the same SoC is for reference only. Always download an entry whose chipset, precision, and QNN backend version exactly match the target environment.

## 1.2 Downloading with MMS

MMS (Model Management Service) lets developers query and download models from a board terminal. Register an account on the [developer registration page](https://auth.aidlux.com/en/register), then sign in:

```bash
mms login
```

List all models or filter by keyword:

```bash
mms list
mms list yolo
```

The output identifies the model name, precision, chipset, and backend version. Use those exact values in the download command:

```bash
# -m: model name  -p: precision  -c: chipset  -b: QNN backend version  -d: download directory
mms get -m yolov6l -p int8 -c qcs8550 -b qnn2.23 -d /home/aidlux/yolov6l
```

For example, download an FP16 QNN 2.36 YOLOv8s package for IQ9/QCS9075:

```bash
mms list | grep YOLOv8s | grep FP16
mms get -m YOLOv8s -p fp16 -c iq9 -b qnn2.36 -d ./
unzip YOLOv8s_iq9_fp16.zip
```

Always use the actual local `mms list` output. Package filenames, internal directory names, and chipset aliases in examples can differ. After extraction, follow the package `README.md` rather than inferring compatibility from its filename.

## 1.3 Running Downloaded Vision Models

Vision-model packages downloaded through MMS generally have this structure:

```text
{model_name}_{SoC}_{precision}/
|-- models/       # Converted model files
|-- code/
|   |-- python/   # AidLite Python examples
|   `-- cpp/      # AidLite C++ examples
`-- README.md
```

For example, the QCS8550 YOLO11s-obb package can use its AidLite Python example for NPU inference:

```bash
cd code
sudo python3 python/run_test.py \
  --target_model ../models/QCS8550/FP16/yolo11s-obb_qcs8550_fp16.qnn236.ctx.bin \
  --imgs python/boats.jpg \
  --invoke_nums 10
```

Model Farm includes models for object detection, classification, monocular depth estimation, segmentation, pose estimation, and oriented bounding-box detection. Example entries include YOLOv8s, ConvNeXt-Tiny, Depth-Anything-V2-Small, FastSAM-S, YOLO11l-Pose, and YOLO11s-obb.

## 1.4 Running Generative Models

AidGen loads generative models: use `aidllm` for language models and `aidmlm` for vision-language models. An MMS package normally contains one or more `.aidem` shards plus cache, embedding, or vision-model files required by its configuration.

Build the language-model example project from the package directory:

```bash
cp -r /usr/local/share/aidgen/examples/aidgen_qnn240/cpp/aidllm/ ./
cd aidllm
mkdir build && cd build
cmake ..
make
```

From the model directory, run the application with the package configuration or a configuration created for that model:

```bash
./aidllm/build/test_aidllm abort ./config.json
```

Available models vary by platform and QNN backend. The source tutorials include examples for MiniCPM5-1B, Qwen3-8B, Meta-Llama-3.1-8B-Instruct, Gemma-2-2B-it, Falcon3-7B-Instruct, DeepSeek-R1-Distill-Qwen-7B, Phi-3.5-mini-instruct, HY-MT1.5-1.8B, and Qwen2.5-VL-3B-Instruct. Shard count, context length, prompt template, and configuration fields vary by model; use the corresponding package README and configuration example.

For HTTP integration, use AidGenSE instead: query models with `aidllm remote-list api`, download one with `aidllm pull api <model-address>`, then start an OpenAI-compatible service with `aidllm start api -m <model-name>`.

## 1.5 Fine-Tuned and Preview Models

For a fine-tuned model, refer to **Model Conversion Reference** on the model detail page or in the package README. Use [AIMO](https://aidlux.com/en/product/aimo) to convert the model to a Qualcomm platform format, then replace the model in the example project with the generated `.amf` file for testing.

Preview models marked **Contact Us** are not available for direct download from the website. Obtain these models through MMS on an APLUX development board and run them with AidLite SDK.

## 1.6 References

- [Model Farm User Guide](https://docs.aidlux.com/en/software/model-farm/model_farm_guide)
- [Quick Start with MMS](https://docs.aidlux.com/en/software/model-farm/model_farm_mms)
- [AIBOX-9075 AI Tutorial](https://community.t-firefly.com/en/docs/products/computers/AIBOX-9075/usage-npu)
