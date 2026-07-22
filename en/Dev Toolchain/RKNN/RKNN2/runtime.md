# On-device Runtime

RKNN2 provides both C/C++ and Python inference APIs on AIBOX-PRO.

## RKNPU2

RKNPU2 provides the C/C++ runtime API for Rockchip NPU platforms. The Linux aarch64 runtime library is located at:

```text
rknpu2/runtime/Linux/librknn_api/aarch64/librknnrt.so
```

If the runtime library is missing or must be updated, deploy the `librknnrt.so` version that matches the device NPU driver to `/usr/lib`. Do not mix a runtime and NPU driver from incompatible SDK releases.

The SDK provides on-device C/C++ inference examples in `rknpu2/examples`.

## RKNN-Toolkit Lite2

RKNN-Toolkit Lite2 provides a Python inference API on the target device. It can only load converted RKNN models and cannot convert models on the device.

Install the basic dependencies first:

```bash
sudo apt-get update
sudo apt-get install -y python3 python3-dev python3-pip gcc python3-opencv python3-numpy
```

Then install the wheel under `rknn-toolkit-lite2/packages` that matches the Python version on the device. For example:

```bash
pip3 install rknn_toolkit_lite2-2.0.0b0-cp310-cp310-linux_aarch64.whl
```

The wheel version, Python ABI, and on-device runtime must be compatible.
