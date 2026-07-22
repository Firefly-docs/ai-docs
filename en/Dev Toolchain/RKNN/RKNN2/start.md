# 1.1 NPU Usage
Platforms such as RK3588 come with a built-in NPU module; the RK3588 NPU offers processing performance of up to 6 TOPS.To use this NPU,you need to download the [RKNN SDK](https://github.com/airockchip/rknn-toolkit2).The RKNN SDK provides C++/Python programming interfaces, which help users deploy RKNN models exported by RKNN-Toolkit2,accelerating the deployment of AI applications. The overall development steps for RKNN are divided into three main parts: model conversion, model evaluation, board-side deployment and execution.

![](./images/npu_development_process_en.png)
* <font color=blue>**Model Conversion**</font>:Supports converting models from `Caffe`、`TensorFlow`、`TensorFlow Lite`、`ONNX`、`DarkNet`、`PyTorch` etc.,to RKNN models.It also supports importing and exporting RKNN models, which can be used on the Rockchip NPU platform.<br></br>
* <font color=blue>**Model Evaluation**</font>:The model evaluation phase helps users quantify and analyze model performance, including key metrics such as accuracy, on-board inference performance, and memory usage.<br></br>
*  <font color=blue>**Board-Side Deployment and Execution**</font>:Involves loading the RKNN model onto the RKNPU platform, and performing model preprocessing, inference, post-processing, and release.<br></br>
