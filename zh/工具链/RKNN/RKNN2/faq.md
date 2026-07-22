# RKNN2 常见问题

## RKNN 模型推理精度低于原始模型

请参考 RKNN SDK 文档中的精度排查章节，依次检查预处理、输入排布、量化数据集、输出解析和算子误差。

## 转换时提示 Expand 算子不支持

尝试将 RKNN-Toolkit2 和 RKNPU2 更新到匹配的较新版本，或在原始模型中使用 `repeat` 等可支持的实现替代 `expand`。

## 板端加载模型失败

确认以下版本互相匹配：

- RKNN-Toolkit2
- `librknnrt.so`
- 板端 NPU 驱动
- 生成 RKNN 模型时配置的目标平台

更多问题请参考 RKNN SDK 中的 `doc` 目录和 `Rockchip_RKNPU_User_Guide_RKNN_SDK` 文档。
