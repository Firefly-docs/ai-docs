# 1. Model Farm 使用指南

[Model Farm](https://aiot.aidlux.com/zh/models) 是 AidLux 对应的模型广场。平台提供针对高通端侧芯片适配和优化的开源模型、可运行的前后处理与推理示例代码，以及在指定硬件上取得的性能指标。浏览模型和性能数据无需登录；下载模型包和示例代码需要开发者账号。

Model Farm 当前支持 Qualcomm QCS6490、QCS8550、QCS8625、Dragonwing IQ9 和 Dragonwing IQ8 等平台。模型下载后可通过 [AidLux](../../工具链/Aidlux/index.md) 的 AidLite、AidGen 或 Qualcomm QNN 运行。

## 1.1 选择模型

在模型广场中可按模型类型、数据精度、芯片平台和关键字筛选。模型详情页包含以下评估信息：

| 字段 | 说明 |
| --- | --- |
| 设备 | 性能测试所用的开发板和芯片。 |
| AI 框架 | 模型转换和推理使用的框架及版本。 |
| 模型数据精度 | 如 FP16、INT8、W4A16。 |
| 推理耗时 | 不包含前处理和后处理的实测耗时。 |
| 精度损失 | 转换模型输出与 FP32 源模型输出的余弦相似度对比。 |
| 模型大小 | 转换后模型文件大小。 |

同一 SoC 在不同开发板上的性能仅供参考。下载时必须选择与设备芯片、精度和 QNN 后端版本完全一致的条目。

## 1.2 使用 MMS 下载

MMS (Model Management Service) 允许在开发板的命令行中查询和下载模型。先在 [开发者账号注册页](https://auth.aidlux.com/zh/register) 注册账号，再登录：

```bash
mms login
```

列出模型，或以关键词筛选：

```bash
mms list
mms list yolo
```

`mms list` 的输出含有模型名称、精度、芯片和后端版本。根据输出精确填写以下参数：

```bash
# -m: 模型名称  -p: 精度  -c: 芯片  -b: QNN 后端版本  -d: 下载目录
mms get -m yolov6l -p int8 -c qcs8550 -b qnn2.23 -d /home/aidlux/yolov6l
```

例如，在 IQ9/QCS9075 平台下载 YOLOv8s 的 FP16、QNN 2.36 包：

```bash
mms list | grep YOLOv8s | grep FP16
mms get -m YOLOv8s -p fp16 -c iq9 -b qnn2.36 -d ./
unzip YOLOv8s_qcs8550_fp16.zip
```

命令中应始终以本机 `mms list` 的实际输出为准。模型包文件名、内部目录名和示例中使用的芯片代号可能不同，解压后请阅读包内 `README.md`，不要仅按文件名推断兼容性。

## 1.3 运行下载的视觉模型

通过 MMS 获得的视觉模型通常具有以下结构：

```text
{model_name}_{SoC}_{precision}/
├── models/       # 转换后的模型文件
├── code/
│   ├── python/   # AidLite Python 示例
│   └── cpp/      # AidLite C++ 示例
└── README.md
```

例如，QCS8550 的 YOLO11s-obb 包可使用其中的 AidLite Python 示例进行 NPU 推理：

```bash
cd code
sudo python3 python/run_test.py \
  --target_model ../models/QCS8550/FP16/yolo11s-obb_qcs8550_fp16.qnn236.ctx.bin \
  --imgs python/boats.jpg \
  --invoke_nums 10
```

Model Farm 中还包含目标检测、分类、单目深度估计、分割、姿态估计和定向边界框检测等模型。典型条目包括 YOLOv8s、ConvNeXt-Tiny、Depth-Anything-V2-Small、FastSAM-S、YOLO11l-Pose 与 YOLO11s-obb。

## 1.4 运行生成式模型

生成式模型由 AidGen 加载：语言模型使用 `aidllm`，视觉语言模型使用 `aidmlm`。MMS 下载后，模型包通常提供一个或多个 `.aidem` 分片和配置所需的缓存、嵌入或视觉模型文件。

以模型包内的示例工程为基础构建语言模型程序：

```bash
cp -r /usr/local/share/aidgen/examples/aidgen_qnn240/cpp/aidllm/ ./
cd aidllm
mkdir build && cd build
cmake ..
make
```

回到模型目录后，用模型包或自行创建的 JSON 配置执行：

```bash
./aidllm/build/test_aidllm abort ./config.json
```

可用的模型因平台和 QNN 后端而异。来源教程中列出了 MiniCPM5-1B、Qwen3-8B、Meta-Llama-3.1-8B-Instruct、Gemma-2-2B-it、Falcon3-7B-Instruct、DeepSeek-R1-Distill-Qwen-7B、Phi-3.5-mini-instruct、HY-MT1.5-1.8B 和 Qwen2.5-VL-3B-Instruct 等示例。每个模型的分片数、上下文长度、提示词模板和配置字段不同，应使用相应模型包的 README 与配置示例。

如果应用通过 HTTP 集成模型，可改用 AidGenSE：使用 `aidllm remote-list api` 查询可下载模型，以 `aidllm pull api <模型地址>` 拉取，再以 `aidllm start api -m <模型名>` 启动 OpenAI 兼容服务。

## 1.5 微调模型和预览模型

对于自行微调的模型，可参考模型详情页“模型转换参考”或代码包 README 中的 **Model Conversion Reference**，使用 [AIMO](https://aidlux.com/product/aimo) 转换为高通平台格式，再将输出的 `.amf` 文件替换至示例工程中测试。

模型广场中标有“联系我们”的 Preview 模型不提供网页直接下载。此类模型需要在阿加犀开发板上通过 MMS 获取，并使用 AidLite SDK 推理。

## 1.6 参考资料

- [Model Farm 用户指南](https://docs.aidlux.com/software/model-farm/model_farm_guide)
- [使用 MMS 快速开始](https://docs.aidlux.com/software/model-farm/model_farm_mms)
- [AIBOX-9075 AI 教程](https://community.t-firefly.com/docs/products/computers/AIBOX-9075/usage-npu)
