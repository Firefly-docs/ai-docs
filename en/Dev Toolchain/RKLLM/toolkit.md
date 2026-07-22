# 2.RKLLM-Toolkit Installation
The RKLLM-Toolkit is currently only available for Linux PC, with `Ubuntu 20.04(x64)` recommended. Since multiple versions of Python environments might be present on the system, it is advisable to use miniforge3 to manage Python environments.
```
# Check if miniforge3 and conda are installed, and if they are, you can skip this section.
conda -V
# Download the miniforge3 installer package.
wget -c https://mirrors.bfsu.edu.cn/github-release/conda-forge/miniforge/LatestRelease/Miniforge3-Linux-x86_64.sh
# Install miniforge3
chmod 777 Miniforge3-Linux-x86_64.sh
bash Miniforge3-Linux-x86_64.sh
# Activate the Conda base environment. The installation directory for miniforge3 will be the default location for Conda.
source ~/miniforge3/bin/activate
# Create a Conda environment named RKLLM-Toolkit with Python 3.8 (recommended version).
conda create -n RKLLM-Toolkit python=3.8
# Enter RKLLM-Toolkit Conda environment
conda activate RKLLM-Toolkit
# Install RKLLM-Toolkit,such as rkllm_toolkit-1.1.4-cp38-cp38-linux_x86_64.whl
pip3 install rkllm-toolkit/rkllm_toolkit-1.1.4-cp38-cp38-linux_x86_64.whl
```
If the following command executes without errors, the installation was successful:
```
python
from rkllm.api import RKLLM
```
