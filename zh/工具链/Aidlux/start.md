# 1. AidLux 与高通 NPU 开发

AidLux 是面向边缘端 AI 应用的开发工具套件，可简化高通平台的运行环境部署，并通过 CPU、GPU 和 NPU 加速模型推理。在 QCS8550 上，NPU 位于 Hexagon DSP 中；文档中出现的 NPU 与 DSP 在此平台通常指同一计算单元。

AidLux 的主要组件如下：

| 组件 | 用途 |
| --- | --- |
| AidLite | AI 执行框架，用于视觉等模型的 CPU、GPU、NPU 推理。 |
| AidGen | 建立在 AidLite 之上的生成式 Transformer 推理框架，提供 LLM 与多模态模型的集成接口。 |
| AidGenSE | 兼容 OpenAI HTTP 协议的生成式 AI 服务，可通过 HTTP 集成到应用。 |
| AidStream | 基于 GStreamer 的实时多媒体处理工具包，适合视频和图像分析流程。 |

可从 [AidLux 文档中心](https://docs.aidlux.com/software/) 获取各组件的完整开发文档。已适配、可下载的模型和示例代码请参阅[模型广场](../../模型广场/Aidlux/index.md)。

## 1.1 QCS8550：AidLux Web Desktop

AIBOX-8550 通常预装 AidLux Web Desktop。先确认容器状态：

```bash
docker ps -a
```

若输出中有名为 `aidlux` 的容器，连接设备的 `eth0` 网络后，通过 `ifconfig eth0` 获取设备 IP，并在局域网浏览器访问 `http://<设备IP>:8000`。默认登录密码为 `aidlux`。

未预装时，请从 [AIBOX-8550 下载页](https://www.t-firefly.com/doc/download/356.html#other_931) 获取 `aidlux_docker.tgz`，上传至设备后安装：

```bash
tar -zxf aidlux_docker.tgz
cd aidlux_docker
chmod +x setup.sh
./setup.sh
```

安装完成后容器会自动启动，后续也会随系统启动。文件浏览器仅能从 `/home/aidlux` 接收上传文件。

### AidLite 示例

Web Desktop 中已提供示例。下面运行 QNN 2.36 的 YOLOv5 Python 示例；参数 `3` 表示使用 NPU (DSP) 加速：

```bash
cd /usr/local/share/aidlite/examples/aidlite_qnn236/python/
sudo python qnn_yolov5_multi.py 3
```

执行成功后，当前目录会生成 `qnn_yolov5_multi.jpg`。

### AidGenSE 服务

先查看本机已有的生成式模型：

```bash
aidllm list api
```

没有安装 AidGenSE 时，安装服务与 UI：

```bash
sudo aid-pkg update
sudo aid-pkg -i aidgense
sudo aidllm install ui
```

列出远端模型、下载并启动一个 QCS8550 适配模型：

```bash
aidllm remote-list api
aidllm pull api aplux/qwen2.5-3B-Instruct-8550
aidllm start api -m qwen2.5-3B-Instruct-8550
```

可使用 `aidllm status api` 查看状态，使用 `aidllm stop api` 停止服务，或在 Web Desktop 中打开 NextChat 进行对话。

## 1.2 AIBOX-9075：Ubuntu 24 原生环境

AIBOX-9075 可在 Ubuntu 24 中配置 AidLux 软件源和依赖。先添加公钥和软件源：

```bash
sudo wget -O- https://archive.aidlux.com/ubuntu24/public.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/private-aidlux.gpg > /dev/null
sudo tee /etc/apt/sources.list.d/private-aidlux.list > /dev/null <<'EOF'
deb [arch=arm64 signed-by=/etc/apt/trusted.gpg.d/private-aidlux.gpg] https://archive.aidlux.com/ubuntu24 noble main
EOF
sudo apt update
```

安装基础组件、AidLite 和 AidGen SDK：

```bash
sudo apt install python3 python3-pip libopencv-dev python3-opencv net-tools
sudo apt install aidlux-aistack-base aidrtcm aid-lms aidlms-sdk cmake aid-mms
sudo apt install qcom-fastrpc1 qcom-fastrpc-dev
sudo apt install aidlite-sdk 'aidlite-*' aidgen-qnn240-sdk libfmt-dev nlohmann-json3-dev
```

安装后应能在 `/usr/local/share/` 中看到 `aidlite` 与 `aidgen` 目录。需要 GPU/OpenCL 支持时，按平台镜像要求安装 Qualcomm Adreno 驱动并创建 `libOpenCL.so` 链接；需要视频流处理时，再安装 `aidstream-gst` 及其 GStreamer 依赖。

## 1.3 选择模型与运行时的注意事项

- 模型必须同时匹配设备芯片、量化精度和 QNN 后端版本。例如 QCS8550 的 `qnn2.29` 包不能假定可用于 IQ9/QCS9075 的 `qnn2.36` 包。
- 下载的模型包通常包括 `models/`、`code/python/`、`code/cpp/` 和 `README.md`。应优先执行包内 README 提供的命令和依赖版本。
- AidLite 适合直接调用视觉模型；AidGen 的 `aidllm` 用于语言模型，`aidmlm` 用于多模态模型；AidGenSE 则用于需要 OpenAI 兼容 HTTP 接口的场景。
- 对模型详情中的性能数据，应将其视为同一 SoC 下对应测试设备的参考值，不同内存、散热和系统配置会造成差异。

## 1.4 参考资料

- [AidLux 软件指南](https://docs.aidlux.com/software/)
- [AIBOX-9075 AI 教程](https://community.t-firefly.com/docs/products/computers/AIBOX-9075/usage-npu)
- [AIBOX-8550 AI 教程](https://community.t-firefly.com/docs/products/computers/AIBOX-8550/usage-npu)
