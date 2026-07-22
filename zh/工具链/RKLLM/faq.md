# RKLLM 常见问题

## 模型转换失败

先检查 PC 的可用内存和磁盘空间。模型参数量越大，转换和量化需要的内存越多。可增加 swapfile，或改用内存更大的 PC。

## 板端无法加载 RKLLM 模型

确认以下组件的版本与目标平台匹配：

- RKLLM-Toolkit
- `.rkllm` 模型的 `target_platform`
- `librkllmrt.so`
- 板端 RKNPU 驱动

## 对话效果异常

检查 tokenizer、BOS/EOS token 和聊天模板是否与原模型匹配。使用空模板或不正确的默认模板可能造成性能和输出质量下降。
