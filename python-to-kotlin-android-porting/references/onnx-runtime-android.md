# Model Inference on Android

Use this reference only when the Python implementation performs ONNX or similar model inference.

## Preserve the model contract

Record:

- model file and version/checksum;
- input and output names;
- tensor shapes, dynamic dimensions, and layout;
- element type and quantization parameters;
- normalization, channel order, padding, resize, and batch behavior;
- execution providers, threads, and graph/runtime options;
- postprocessing thresholds, class mappings, and coordinate transforms.

The Kotlin side must use the Android runtime version and APIs supported by the target project. Do not assume that Python provider settings, operators, or default threading behavior are identical on Android.

## Android concerns

- Load model and label assets through Android resources/assets or the project's established file provider.
- Avoid doing model initialization or inference on the main thread.
- Define session ownership and close/release native resources where the runtime requires it.
- Reuse input/output buffers only when concurrent calls cannot corrupt them.
- Check model size, memory pressure, startup latency, and supported ABIs.
- Compare preprocessed input tensors before comparing final predictions; this localizes layout and normalization errors.

## Validation sequence

1. Compare model file identity.
2. Compare prepared input tensor shape, dtype, and representative values.
3. Compare raw output tensors with numerical tolerances.
4. Compare postprocessed results, labels, boxes, masks, and scores.
5. Exercise lifecycle, repeated calls, concurrent calls if supported, and error paths on Android.

If the runtime or provider differs, record the expected numerical variation and validate downstream decisions rather than hiding differences with broad tolerances.
