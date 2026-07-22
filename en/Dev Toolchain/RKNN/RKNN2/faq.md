# RKNN2 FAQ

## The RKNN model is less accurate than the source model

Follow the accuracy troubleshooting section in the RKNN SDK documentation. Check preprocessing, input layout, the quantization dataset, output parsing, and operator errors in sequence.

## Model conversion reports that the Expand operator is unsupported

Update RKNN-Toolkit2 and RKNPU2 to compatible newer versions, or replace `expand` in the source model with a supported implementation such as `repeat`.

## The model cannot be loaded on the target device

Verify that the following components are mutually compatible:

- RKNN-Toolkit2
- `librknnrt.so`
- The NPU driver on the target device
- The target platform selected when the RKNN model was generated

For more troubleshooting information, see the `doc` directory in the RKNN SDK and the `Rockchip_RKNPU_User_Guide_RKNN_SDK` document.
