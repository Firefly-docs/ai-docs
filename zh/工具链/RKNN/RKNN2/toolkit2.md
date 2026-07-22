# RKNN-Toolkit2

RKNN-Toolkit2 是运行在 x86_64 PC 上的 Python 开发套件，用于模型转换、推理模拟和性能评估。

## 安装

RKNN SDK 支持 Docker 和 Python wheel 两种安装方式。以 Ubuntu 20.04 x86_64 和 Python 3.8 为例：

```bash
conda create -n RKNN-Toolkit2 python=3.8
conda activate RKNN-Toolkit2

pip3 install -r rknn-toolkit2/packages/requirements_cp38-2.0.0b0.txt
pip3 install rknn-toolkit2/packages/rknn_toolkit2-2.0.0b0+9bab5682-cp38-cp38-linux_x86_64.whl
```

wheel 文件名和依赖文件路径会随 SDK 版本变化，安装时应以已下载 SDK 中的实际文件为准。

验证安装：

```python
from rknn.api import RKNN
```

更多安装方式请参考 [RKNN-Toolkit2 文档](https://github.com/airockchip/rknn-toolkit2/tree/master/doc)。

## 模型转换

`rknn-toolkit2/examples` 提供了多种模型转换示例。典型的 ONNX 模型转换流程如下：

```python
from rknn.api import RKNN

rknn = RKNN(verbose=True)
rknn.config(target_platform="rk3588")
rknn.load_onnx(model="model.onnx")
rknn.build(do_quantization=True, dataset="dataset.txt")
rknn.export_rknn("model.rknn")
rknn.release()
```

实际开发中需要根据模型输入、均值、归一化参数和目标芯片调整 `config()` 配置。如果 SDK 示例未包含所需模型，可参考 [RKNN Model Zoo](https://github.com/airockchip/rknn_model_zoo)。
