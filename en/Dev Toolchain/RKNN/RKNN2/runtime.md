# 1.3 RKNPU2 Usage
RKNPU2 provides C/C++ programming interfaces for model inference on the `board-side Rockchip NPU platform`.
#### 1.3.1 Environment Installation
If the RKNPU2 Runtime library file `librknnrt.so` is missing or needs updating on the board, for Linux systems, you can push `rknpu2/runtime/Linux/librknn_api/aarch64/librknnrt.so` to the `/usr/lib` directory using `scp`.
#### 1.3.2 Board-Side Inference
The RKNN SDK directory `rknpu2/examples` provides many model inference demos. Users can refer to these examples to develop and deploy their own AI applications.The [rknn_model_zoo](https://github.com/airockchip/rknn_model_zoo) repository also provides C/C++ examples.

### 1.4 RKNN-Toolkit Lite2 Introduction
RKNN-Toolkit Lite2 provides a `Python` interface for board-side model inference, making it convenient for users to develop AI applications using Python. **Note**: RKNN-Toolkit Lite2 is installed and run on the `board-side`, and it is used only for inference; it does not support model conversion.

#### 1.4.1 RKNN-Toolkit Lite2 Installation
```
# Install python3/pip3
sudo apt-get update
sudo apt-get install -y python3 python3-dev python3-pip gcc python3-opencv python3-numpy
# Install RKNN-Toolkit Lite2; find the installation package in rknn-toolkit-lite2/packages and install the corresponding version according to the system python version
pip3 install rknn_toolkit_lite2-2.0.0b0-cp310-cp310-linux_aarch64.whl

```
#### 1.4.2 RKNN-Toolkit Lite2 Usage
In the RKNN SDK/rknn-toolkit-lite2/examples directory, there are applications developed based on RKNN-Toolkit Lite2. Although the number of provided examples is limited, in practice, the interfaces of `RKNN-Toolkit Lite2` and `RKNN-Toolkit2` are quite similar. Users can refer to the RKNN-Toolkit2 examples for porting to RKNN-Toolkit Lite2.

### 1.5 Detailed Development Documentation
For detailed usage of NPU and Toolkit, please refer to the `doc` documentation under RKNN SDK.
