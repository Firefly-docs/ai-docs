# RKLLM FAQ

## Model conversion fails

Check the available system RAM and disk space. Larger models require more memory during conversion and quantization. Increase the swapfile size or use a PC with more memory.

## The target cannot load the RKLLM model

Verify that the following components match the target platform and are mutually compatible:

- RKLLM-Toolkit
- The `target_platform` used by the `.rkllm` model
- `librkllmrt.so`
- The RKNPU driver on the target device

## Chat output is abnormal

Check that the tokenizer, BOS/EOS tokens, and chat template match the source model. An empty or incorrect default template can reduce performance and output quality.
