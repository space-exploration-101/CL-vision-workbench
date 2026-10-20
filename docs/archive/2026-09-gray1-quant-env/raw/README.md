# Gray1 quantization image

This isolated image derives from the accepted gray1 training/export image and
adds a frozen copy of the Quantitize pipeline and its local ONNX Runtime
patches. It does not replace or retag `quantitize-platform-api:latest`.

Build context layout on H200:

```text
/data3/ywang/yolo-gray1-quant-env/
  docker/Dockerfile
  docker/verify_gray1_quant_runtime.py
  src/quantitize-platform/pipeline/
  src/quantitize-platform/patches/
```

The image is an environment-alignment artifact. Gray1 calibration, FPGA input
roundtrip, bin generation, and ABI acceptance belong to runbook Step 14.
