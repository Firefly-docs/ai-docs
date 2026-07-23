# 1. AidLux and Qualcomm NPU Development

AidLux is an edge AI development toolkit that simplifies environment deployment on Qualcomm platforms and accelerates model inference with CPU, GPU, and NPU compute. On QCS8550, the NPU is part of the Hexagon DSP; in this context, NPU and DSP generally refer to the same compute unit.

AidLux consists of the following primary components:

| Component | Purpose |
| --- | --- |
| AidLite | AI execution framework for CPU, GPU, and NPU inference of vision and other models. |
| AidGen | Generative Transformer inference framework built on AidLite, with interfaces for LLMs and multimodal models. |
| AidGenSE | Generative AI service compatible with the OpenAI HTTP protocol. |
| AidStream | GStreamer-based real-time multimedia processing toolkit for video and image analysis pipelines. |

See the [AidLux Documentation Center](https://docs.aidlux.com/en/software/) for complete component documentation. For optimized models and runnable example code, see the [AidLux Model Farm](../../Model%20Zoo/Aidlux/index.md).

## 1.1 QCS8550: AidLux Web Desktop

AIBOX-8550 usually includes AidLux Web Desktop. Verify that its container exists:

```bash
docker ps -a
```

If the output includes a container named `aidlux`, connect the device through `eth0`, obtain its IP address with `ifconfig eth0`, then open `http://<device-ip>:8000` from a browser on the same LAN. The default login password is `aidlux`.

If it is not installed, download `aidlux_docker.tgz` from the [AIBOX-8550 download page](https://en.t-firefly.com/doc/download/366.html#other_822), transfer it to the device, and install it:

```bash
tar -zxf aidlux_docker.tgz
cd aidlux_docker
chmod +x setup.sh
./setup.sh
```

The container starts automatically after installation and on subsequent boots. The file browser accepts uploads only in `/home/aidlux`.

### AidLite example

Web Desktop includes example applications. The following command runs the QNN 2.36 YOLOv5 Python example; the argument `3` selects NPU (DSP) acceleration:

```bash
cd /usr/local/share/aidlite/examples/aidlite_qnn236/python/
sudo python qnn_yolov5_multi.py 3
```

The command creates `qnn_yolov5_multi.jpg` in the current directory.

### AidGenSE service

First, list locally available generative models:

```bash
aidllm list api
```

If AidGenSE is not installed, install the service and its UI:

```bash
sudo aid-pkg update
sudo aid-pkg -i aidgense
sudo aidllm install ui
```

List remote models, download a QCS8550-compatible model, and start it:

```bash
aidllm remote-list api
aidllm pull api aplux/qwen2.5-3B-Instruct-8550
aidllm start api -m qwen2.5-3B-Instruct-8550
```

Use `aidllm status api` to inspect the service and `aidllm stop api` to stop it. NextChat in Web Desktop can be used as a chat client.

## 1.2 AIBOX-9075: Native Ubuntu 24 Environment

On AIBOX-9075, configure the AidLux repository and dependencies in Ubuntu 24. Add the public key and package source first:

```bash
sudo wget -O- https://archive.aidlux.com/ubuntu24/public.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/private-aidlux.gpg > /dev/null
sudo tee /etc/apt/sources.list.d/private-aidlux.list > /dev/null <<'EOF'
deb [arch=arm64 signed-by=/etc/apt/trusted.gpg.d/private-aidlux.gpg] https://archive.aidlux.com/ubuntu24 noble main
EOF
sudo apt update
```

Install the base components and AidLite and AidGen SDKs:

```bash
sudo apt install python3 python3-pip libopencv-dev python3-opencv net-tools
sudo apt install aidlux-aistack-base aidrtcm aid-lms aidlms-sdk cmake aid-mms
sudo apt install qcom-fastrpc1 qcom-fastrpc-dev
sudo apt install aidlite-sdk 'aidlite-*' aidgen-qnn240-sdk libfmt-dev nlohmann-json3-dev
```

After installation, `/usr/local/share/` should contain `aidlite` and `aidgen`. For GPU/OpenCL support, install the Qualcomm Adreno driver and create the `libOpenCL.so` link as required by the platform image. Install `aidstream-gst` and its GStreamer dependencies separately when a video-streaming pipeline is needed.

## 1.3 Model and Runtime Compatibility

- A model package must match the device chipset, quantization precision, and QNN backend version. For example, a QCS8550 `qnn2.29` package must not be assumed compatible with an IQ9/QCS9075 `qnn2.36` package.
- Downloaded packages typically contain `models/`, `code/python/`, `code/cpp/`, and `README.md`. Use the commands and dependency versions documented in the package README.
- Use AidLite for direct vision-model integration. In AidGen, `aidllm` serves language models and `aidmlm` serves multimodal models. Use AidGenSE when an OpenAI-compatible HTTP interface is required.
- Model Farm performance figures are references from the stated test device. Memory, cooling, and system configuration can affect results on another board with the same SoC.

## 1.4 References

- [AidLux Software Guide](https://docs.aidlux.com/en/software/)
- [AIBOX-9075 AI Tutorial](https://community.t-firefly.com/en/docs/products/computers/AIBOX-9075/usage-npu)
- [AIBOX-8550 AI Tutorial](https://community.t-firefly.com/en/docs/products/computers/AIBOX-8550/usage-npu)
