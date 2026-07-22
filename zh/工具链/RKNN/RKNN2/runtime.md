# 1.3 RKNPU2使用
RKNPU2 提供在`板端Rockchip NPU 平台`上进行模型推理的C/C++编程接口。
#### 1.3.1 环境安装
RKNPU2 Runtime 库文件`librknnrt.so`，若板端缺少该文件或需要更新，对于Linux系统可以将`rknpu2/runtime/Linux/librknn_api/aarch64/librknnrt.so`通过`scp`推送至`/usr/lib`目录。
#### 1.3.2 板端推理
在RKNN SDK 目录`rknpu2/examples`提供了许多模型推理Demo，用户可以参考例程开发部署自己的AI应用。[rknn_model_zoo](https://github.com/airockchip/rknn_model_zoo)仓库同样提供了C/C++的例程

### 1.4 RKNN-Toolkit Lite2介绍
RKNN-Toolkit Lite2 为用户提供`板端模型推理`的 `Python` 接口，方便用户使用 Python 语言进行 AI 应用开发。**注意**：RKNN-Toolkit Lite2安装运行在`板端`，且仅作推理使用，无法转换模型。
#### 1.4.1 RKNN-Toolkit Lite2安装
```
# 安装python3/pip3
sudo apt-get update
sudo apt-get install -y python3 python3-dev python3-pip gcc python3-opencv python3-numpy
# RKNN-Toolkit Lite2安装，在rknn-toolkit-lite2/packages找到安装包，根据系统python版本安装对应版本
pip3 install rknn_toolkit_lite2-2.0.0b0-cp310-cp310-linux_aarch64.whl
```
#### 1.4.2 RKNN-Toolkit Lite2使用
在 RKNN SDK/rknn-toolkit-lite2/examples 目录，有基于RKNN-Toolkit Lite2 开发的应用，虽然提供的例程数量有限，但是实际上`RKNN-Toolkit Lite2` 和 `RKNN-Toolkit2`的接口使用是十分类似的，用户完全可以参考RKNN-Toolkit2例程移植到RKNN-Toolkit Lite2。

### 1.5 详细开发文档
NPU 及 Toolkit 详细使用方法请参考RKNN SDK下`doc`文档。
