# NumPy and OpenCV to Android

Use this reference when the Python implementation uses NumPy, OpenCV, or array/image operations. Verify exact behavior against fixtures because similarly named APIs can differ in layout, dtype, interpolation, border handling, or channel order.

## Array contract

Record for every array or image:

- shape and axis meaning;
- dtype and valid range;
- row-major/column-major assumptions;
- channel order and color space;
- ownership, mutability, and row stride;
- treatment of empty arrays and non-contiguous views.

Possible Android representations include `FloatArray`, `IntArray`, `ByteArray`, `ByteBuffer`, OpenCV `Mat`, `Bitmap`, or a project-specific tensor type. Select one based on the operation sequence and runtime constraints, not only on API similarity.

## Frequent divergence sources

- RGB versus BGR channel order;
- alpha premultiplication in `Bitmap`;
- signed versus unsigned byte interpretation;
- `uint8` arithmetic versus Kotlin byte/int arithmetic;
- integer rounding and clipping;
- `reshape` versus `resize` semantics;
- inclusive/exclusive rectangle bounds;
- row/column indexing order;
- interpolation kernel and border mode;
- image orientation metadata;
- copying a view versus sharing the underlying buffer.

## Implementation guidance

- Normalize input format once at the boundary and document it.
- Avoid repeated conversions between `Bitmap`, `Mat`, and byte arrays in inner loops.
- Reuse buffers only when ownership and thread safety are clear.
- Keep shape and stride checks close to the operation that requires them.
- Test one-pixel and one-row edge cases, odd dimensions, empty masks, and maximum channel values.
- If a native OpenCV dependency is used, verify supported Android ABIs and the exact runtime version in the target project.

Do not replace a NumPy/OpenCV operation with a visually similar shortcut without measuring the effect on downstream outputs.
