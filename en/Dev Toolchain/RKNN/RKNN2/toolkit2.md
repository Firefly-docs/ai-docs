# RKNN-Toolkit2

RKNN-Toolkit2 is a Python development package that runs on an x86_64 PC. It provides model conversion, simulated inference, and performance evaluation.

## Installation

The RKNN SDK supports both Docker and Python wheel installation. The following example uses Ubuntu 20.04 x86_64 and Python 3.8:

```bash
conda create -n RKNN-Toolkit2 python=3.8
conda activate RKNN-Toolkit2

pip3 install -r rknn-toolkit2/packages/requirements_cp38-2.0.0b0.txt
pip3 install rknn-toolkit2/packages/rknn_toolkit2-2.0.0b0+9bab5682-cp38-cp38-linux_x86_64.whl
```

Wheel names and dependency file paths change between SDK releases. Use the actual files included in the downloaded SDK.

Verify the installation:

```python
from rknn.api import RKNN
```

For other installation methods, see the [RKNN-Toolkit2 documentation](https://github.com/airockchip/rknn-toolkit2/tree/master/doc).

## Model Conversion

The `rknn-toolkit2/examples` directory contains conversion examples for multiple models. A typical ONNX conversion workflow is shown below:

```python
from rknn.api import RKNN

rknn = RKNN(verbose=True)
rknn.config(target_platform="rk3588")
rknn.load_onnx(model="model.onnx")
rknn.build(do_quantization=True, dataset="dataset.txt")
rknn.export_rknn("model.rknn")
rknn.release()
```

For a real model, adjust the `config()` settings to match its inputs, mean values, normalization parameters, and target platform. If the SDK does not include the required example, refer to the [RKNN Model Zoo](https://github.com/airockchip/rknn_model_zoo).
