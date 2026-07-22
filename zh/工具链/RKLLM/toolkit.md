# 2.RKLLM-Toolkit 安装
RKLLM-Toolkit目前只适用于Linux PC，建议使用`Ubuntu20.04(x64)`。因为系统中可能同时有多个版本的 Python 环境，建议使用 miniforge3 管理 Python 环境。
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
# 创建一个 Python3.8 版本（建议版本）名为 RKLLM-Toolkit 的 Conda 环境
conda create -n RKLLM-Toolkit python=3.8
# 进入 RKLLM-Toolkit Conda 环境
conda activate RKLLM-Toolkit
# 安装 RKLLM-Toolkit，如rkllm_toolkit-1.1.4-cp38-cp38-linux_x86_64.whl
pip3 install rkllm-toolkit/rkllm_toolkit-1.1.4-cp38-cp38-linux_x86_64.whl
```
若执行以下命令没有报错，则安装成功。
```
python
from rkllm.api import RKLLM
```
