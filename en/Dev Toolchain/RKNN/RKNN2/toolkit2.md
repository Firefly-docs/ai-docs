# 1.2 RKNN-Toolkit2
RKNN-Toolkit2 RKNN-Toolkit2 is a development suite provided for model conversion, inference, and performance evaluation on `PC` and Rockchip NPU platforms. Users can conveniently perform various operations using the Python interface provided by this tool.**Note**:The RKNN-Toolkit2 development suite runs on the `PC x86_64` platform and should not be installed on the target device.

#### 1.2.1 RKNN-Toolkit2 Installation
The RKNN SDK offers two installation methods for RKNN-Toolkit2: via Docker and via pip. Users can choose either method for installation. Here is an example using pip on `PC Ubuntu20.04(x64)`.Since there may be multiple versions of Python environments on the system, it is recommended to use miniforge3 to manage the Python environment.
```
# Check if miniforge3 and conda are installed. If already installed, this step can be skipped.
conda -V
# Download the miniforge3 installation package
wget -c https://mirrors.bfsu.edu.cn/github-release/conda-forge/miniforge/LatestRelease/Miniforge3-Linux-x86_64.sh
# Install miniforge3
chmod 777 Miniforge3-Linux-x86_64.sh
bash Miniforge3-Linux-x86_64.sh
# Enter the Conda base environment; miniforge3 is the installation directory
source ~/miniforge3/bin/activate
# Create a Conda environment named RKNN-Toolkit2 with Python 3.8 (recommended version)
conda create -n RKNN-Toolkit2 python=3.8
# Enter the RKNN-Toolkit2 Conda environment
conda activate RKNN-Toolkit2
# Install dependency libraries
pip3 install -r rknn-toolkit2/packages/requirements_cp38-2.0.0b0.txt
# Install RKNN-Toolkit2, for example:
pip3 install rknn-toolkit2/packages/rknn_toolkit2-2.0.0b0+9bab5682-cp38-cp38-linux_x86_64.whl
```
If no errors are reported when executing the following command, the installation is successful.
```
python
from rknn.api import RKNN
```
If installation fails or if other installation methods are needed, please refer to the [RKNN SDK documentation](https://github.com/airockchip/rknn-toolkit2/tree/master/doc).

#### 1.2.2 Model Conversion Demo
In the rknn-toolkit2/examples directory, there are various function demos. Here, we will run a model conversion demo as an example. This demo shows the process of converting a yolov5 onnx model to an RKNN model on a PC, then exporting and inferring on a simulator. For the specific implementation of model conversion, refer to the demo's source code and the RKNN SDK documentation.
```
root@9893c1c48f42:/rknn-toolkit2/examples/onnx/yolov5# python3 test.py 
I rknn-toolkit2 version: 2.0.0b0+9bab5682
--> Config model
done
--> Loading model
I It is recommended onnx opset 19, but your onnx model opset is 12!
I Model converted from pytorch, 'opset_version' should be set 19 in torch.onnx.export for successful convert!
I Loading : 100%|██████████████████████████████████████████████| 125/125 [00:00<00:00, 22152.70it/s]
done
--> Building model
I GraphPreparing : 100%|███████████████████████████████████████| 149/149 [00:00<00:00, 10094.68it/s]
I Quantizating : 100%|███████████████████████████████████████████| 149/149 [00:00<00:00, 428.06it/s]
W build: The default input dtype of 'images' is changed from 'float32' to 'int8' in rknn model for performance!
                       Please take care of this change when deploy rknn model with Runtime API!                                                                                                                      
W build: The default output dtype of 'output' is changed from 'float32' to 'int8' in rknn model for performance!
                      Please take care of this change when deploy rknn model with Runtime API!                                                                                                                       
W build: The default output dtype of '283' is changed from 'float32' to 'int8' in rknn model for performance!
                      Please take care of this change when deploy rknn model with Runtime API!                                                                                                                       
W build: The default output dtype of '285' is changed from 'float32' to 'int8' in rknn model for performance!
                      Please take care of this change when deploy rknn model with Runtime API!                                                                                                                       
I rknn building ...
I rknn buiding done.
done
--> Export rknn model
done
--> Init runtime environment
I Target is None, use simulator!
done
--> Running model
I GraphPreparing : 100%|███████████████████████████████████████| 153/153 [00:00<00:00, 10289.55it/s]
I SessionPreparing : 100%|██████████████████████████████████████| 153/153 [00:00<00:00, 1926.55it/s]
done
   class        score      xmin, ymin, xmax, ymax
--------------------------------------------------
   person       0.884     [ 208,  244,  286,  506]
   person       0.868     [ 478,  236,  559,  528]
   person       0.825     [ 110,  238,  230,  533]
   person       0.334     [  79,  353,  122,  517]
    bus         0.705     [  92,  128,  554,  467]
Save results to result.jpg!
```
If the rknn-toolkit2 is missing a model demo you need, you can refer to the [rknn_model_zoo](https://github.com/airockchip/rknn_model_zoo) repository for Python examples.
