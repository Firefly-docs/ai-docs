# 1.2 RKNN-Toolkit2工具
RKNN-Toolkit2 是为用户提供在`PC`、Rockchip NPU 平台上进行模型转换、推理和性能评估的开发套件，用户通过该工具提供的 Python 接口可以便捷地完成各种操作。**注意**：RKNN-Toolkit2 开发套件运行在`PC x86_64`平台上，请勿安装在目标板端。

#### 1.2.1 RKNN-Toolkit2安装
RKNN SDK 提供两种RKNN-Toolkit2安装方式，通过Docker方式安装和通过pip方式安装，用户可自行选择任意一种方式进行安装。这里以pip方式为例，在`PC Ubuntu20.04(x64)`安装。因为系统中可能同时有多个版本的 Python 环境，建议使用 miniforge3 管理 Python 环境。
```
# 检查是否安装 miniforge3 和 conda 版本信息，若已安装则可省略此小节步骤。
conda -V
# 下载 miniforge3 安装包
wget -c https://mirrors.bfsu.edu.cn/github-release/conda-forge/miniforge/LatestRelease/Miniforge3-Linux-x86_64.sh
# 安装 miniforge3
chmod 777 Miniforge3-Linux-x86_64.sh
bash Miniforge3-Linux-x86_64.sh
# 进入Conda base 环境，miniforge3 为安装目录
source ~/miniforge3/bin/activate
# 创建一个 Python3.8 版本（建议版本）名为RKNN-Toolkit2 的 Conda 环境
conda create -n RKNN-Toolkit2 python=3.8
# 进入 RKNN-Toolkit2 Conda 环境
conda activate RKNN-Toolkit2
# 安装依赖库
pip3 install -r rknn-toolkit2/packages/requirements_cp38-2.0.0b0.txt
# 安装 RKNN-Toolkit2，如
pip3 install rknn-toolkit2/packages/rknn_toolkit2-2.0.0b0+9bab5682-cp38-cp38-linux_x86_64.whl
```
若执行以下命令没有报错，则安装成功。
```
python
from rknn.api import RKNN
```
若安装失败或需要其他安装方式请查阅[RKNN SDK文档](https://github.com/airockchip/rknn-toolkit2/tree/master/doc)。

#### 1.2.2 模型转换Demo
在rknn-toolkit2/examples下有各种功能的 Demo ，这里我们运行一个模型转换 Demo 为例子，这个 Demo 展示了在 PC 上将 yolov5 onnx 模型转换成 RKNN 模型，然后导出、在仿真器上推理的过程。模型转换的具体实现请参考 Demo 内源代码以及RKNN SDK文档。
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
如果 rknn-toolkit2 缺少您需要的模型Demo，可以参考[rknn_model_zoo](https://github.com/airockchip/rknn_model_zoo)仓库的python例程。
