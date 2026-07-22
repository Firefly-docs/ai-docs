# RK1820/RK1828 平台

AIBOX-PRO 的 RKNN3 开发链由 RK3588 主控、RK1820/RK1828 协处理器和 PCIe 通信链路组成：

- **RK3588 主控（Host）**：负责任务调度、资源分配和整体控制。
- **RK1820/RK1828 协处理器（Device）**：作为 AI 计算加速单元，执行神经网络推理。
- **PCIe 接口**：实现主控与协处理器之间的低延迟、高带宽数据交互。

## rknn-smi

`rknn-smi` 用于 RK1820/RK1828 设备信息收集、功能配置、状态监控和日志管理。

```bash
# 查询软件版本
sudo rknn-smi -v

# 查询硬件信息
sudo rknn-smi info -l

# 持续监控状态
sudo rknn-smi info -w

# 设置性能模式
sudo rknn-smi set -t work_mode -s 2
```

AIBOX-PRO 硬件不支持通过 `rknn-smi info -t power` 查询功耗。详细用法请参考 RK182X SDK 中的 `docs/Tools/Rockchip_User_Guide_RKNN-SMI_Tool_CN.pdf`。

### 概率性出现 Failed to initialize rknnsmi

可在 `/lib/systemd/system/rknn3.service` 中为 Runtime 启动增加适当延迟：

```ini
[Unit]
Description=rknn3 runtime service
DefaultDependencies=no
After=local-fs.target

[Service]
Type=forking
ExecStartPre=/bin/sleep 3
ExecStart=/bin/rknn3_startup start
ExecStop=/bin/rknn3_startup stop

[Install]
WantedBy=sysinit.target
```

修改系统服务前建议先确认问题确实由设备初始化时序引起。

## RKNN3 SDK

RK182X SDK 中的 RKNN3 目录结构如下：

```text
rknn/
├── rknn3-model-zoo
├── rknn3-runtime
├── rknn3-toolkit
└── rknn-gstreamer-plugins
```

![RKNN3 SDK 框图](./images/RKNN3-SDK-Block-Diagram.png)

### RKNN3 Model Zoo

RKNN3 Model Zoo 提供 RK1820/RK1828 平台经典模型的转换与部署示例：

- [rknn3-model-zoo](https://github.com/airockchip/rknn3-model-zoo)

### RKNN3 Runtime

RKNN3 Runtime 提供 C API。开发者可在 RK3588 Host 端使用 C/C++ 应用程序调用 RK1820/RK1828 执行模型推理。

### RKNN3 Toolkit

RKNN3 Toolkit 运行在 PC 平台，用于模型转换、推理测试和性能评估。

RKNN3 Toolkit 与 [RKNN-Toolkit](https://github.com/airockchip/rknn-toolkit) 及 [RKNN-Toolkit2](https://github.com/airockchip/rknn-toolkit2) **不兼容**。

- [rknn3-toolkit](https://github.com/airockchip/rknn3-toolkit)
