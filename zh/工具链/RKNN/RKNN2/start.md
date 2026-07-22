# 1.1 NPU使用
RK3588 等主控内置 NPU 模块，其中 RK3588 NPU 处理性能最高可达6TOPS。使用该NPU需要下载[RKNN SDK](https://github.com/airockchip/rknn-toolkit2)，RKNN SDK 提供 C++/python 编程接口，能够帮助用户部署使用 RKNN-Toolkit2 导出的 RKNN 模型，加速 AI 应用的落地。RKNN 的整体开发步骤，流程主要分为3 个部分：模型转换、模型评估和板端部署运行。
![](./images/npu_development_process.png)
* <font color=blue>**模型转换**</font>: 支持 `Caffe`、`TensorFlow`、`TensorFlow Lite`、`ONNX`、`DarkNet`、`PyTorch` 等模型转为 RKNN 模型，并支持 RKNN 模型导入导出，RKNN 模型能够在 Rockchip NPU 平台上加载使用。<br></br>
* <font color=blue>**模型评估**</font>: 模型评估阶段帮助用户量化和分析模型性能，包括精度、连板推理性能和内存占用等关键指标。<br></br>
*  <font color=blue>**板端部署运行**</font>: 加载 RKNN 模型到 RKNPU 平台，进行模型前处理、推理、后处理、释放等流程。<br></br>
