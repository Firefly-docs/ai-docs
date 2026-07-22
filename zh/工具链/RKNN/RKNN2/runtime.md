# 板端运行环境

RKNN2 在 AIBOX-PRO 板端提供 C/C++ 和 Python 两种推理接口。

## RKNPU2

RKNPU2 提供 Rockchip NPU 平台的 C/C++ Runtime API。Linux aarch64 Runtime 库位于 SDK 的：

```text
rknpu2/runtime/Linux/librknn_api/aarch64/librknnrt.so
```

如果板端缺少 Runtime 库或需要更新，可将与板端 NPU 驱动匹配的 `librknnrt.so` 部署到 `/usr/lib`。不建议混用不同 SDK 版本的 Runtime 与 NPU 驱动。

SDK 的 `rknpu2/examples` 目录提供了板端 C/C++ 推理示例。

## RKNN-Toolkit Lite2

RKNN-Toolkit Lite2 提供板端 Python 推理接口。它只能加载已转换的 RKNN 模型，不支持在板端转换模型。

先安装基础依赖：

```bash
sudo apt-get update
sudo apt-get install -y python3 python3-dev python3-pip gcc python3-opencv python3-numpy
```

再根据板端 Python 版本安装 SDK `rknn-toolkit-lite2/packages` 中对应的 wheel，例如：

```bash
pip3 install rknn_toolkit_lite2-2.0.0b0-cp310-cp310-linux_aarch64.whl
```

wheel 版本、Python ABI 和板端 Runtime 应保持匹配。
