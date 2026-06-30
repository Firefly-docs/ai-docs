# 安装指南

这里是一段安装说明文本。

## 依赖安装

```bash
sudo apt update && sudo apt install -y \
    测试包名一 \
    测试包名二 \
    测试包名三
```

## 源码编译

```bash
git clone https://这里是一个.git地址.com/测试仓库
cd 测试仓库
mkdir 编译目录 && cd 编译目录
cmake .. -D测试参数=Release
make -j$(nproc)
```

## 包管理器安装

```bash
pip install 测试包名
```

## 验证安装

```bash
测试命令 --version
```

预期输出：
```
测试工具 版本 v9.9.9
```

## 卸载

```bash
sudo apt remove 测试包名
# 或者
pip uninstall 测试包名
```
