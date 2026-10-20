# FPGA Layer Trace

This directory contains one image's intermediate tensors from the Pose21 Gray1
weight-only ONNX model. `manifest.json` is the authoritative index.

Activation BIN files are little-endian float16. NCHW tensors use T32 channel
packing: `[1,C,H,W] -> [ceil(C/32),H,W,32]`. The decoded model output uses
`[1,C,N] -> [N,ceil(C/32),32]`; the single-tile `[1,31,34000]` output is stored
as `[34000,32]`. Padding lanes are zero and must not enter post-processing.

Compute layers retain the established `T32_Lxx_sxx` IDs. Routing-only records
use `T32_Lxx_rxx` so Concat and head decode tensors do not renumber weight/BN
layers. All inputs of multi-input operators are listed separately.
