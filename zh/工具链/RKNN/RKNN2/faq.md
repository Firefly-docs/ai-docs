# 1.6 FAQs
**Q1：rknn模型推理精度对比原模型精度下降？**

A1：请参考《Rockchip_RKNPU_User_Guide_RKNN_SDK*》文档的精度排查章节逐步查找原因。

**Q2：转换时提示 Expand 算子不支持？**

A2：尝试更新 RKNN-Toolkit2 / RKNPU2 至最新版本或者采用 repeat 算子来替代 expand 算子。

**更多转换问题或者报错原因可以参考《Rockchip_RKNPU_User_Guide_RKNN_SDK\*》文档常见问题章节**。
