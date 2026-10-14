---
name: python-to-kotlin-android-porting
description: Port Python algorithms or inference pipelines to Kotlin for Android applications and libraries, while preserving data semantics and validating behavior against the Python implementation. Use when Python code needs to become maintainable Android/Kotlin code; do not use for simple syntax translation, Android AAR packaging alone, or Python server code that will remain on the backend.
metadata:
  short-description: Port Python algorithms to Kotlin Android
---

# Python to Kotlin Android Porting

Use this skill to migrate a Python algorithm, preprocessing pipeline, inference flow, or utility module into Kotlin code that runs in an Android app or Android Library. Treat the Python implementation as a behavioral reference, not as text to translate line by line.

The goal is a maintainable Android implementation with explicit input/output contracts, equivalent numerical and image semantics where required, Android-appropriate dependencies, and evidence of parity. A successful Kotlin build alone is not proof of algorithm equivalence.

## Boundaries

- This skill can modify source code when the user requests a migration or implementation. Preserve unrelated changes and keep the migration scoped to the requested modules.
- Do not assume every Python construct has a direct Android equivalent. Flag dynamic typing, reflection, multiprocessing, filesystem/network services, training code, server-only dependencies, and unsupported scientific libraries for redesign or confirmation.
- Do not silently replace an algorithm with a simpler approximation. If an exact dependency or operation cannot be reproduced, document the difference and obtain direction when it materially affects results.
- Do not package or publish an AAR as part of ordinary migration unless requested. After migration, an Android AAR packaging workflow may build and verify the library artifact.
- Do not claim device runtime support from JVM tests alone, or claim Python/Kotlin parity from compilation alone.

## Workflow

### 1. Discover the source and target

Inspect the Python implementation and the Android repository before editing:

- identify the true Python entry points, call graph, configuration, constants, and supported Python/runtime versions;
- identify input formats, output fields, shapes, dtypes, value ranges, coordinate systems, channel order, color space, units, and error behavior;
- inventory imports and classify them as standard library, portable pure Python, numerical/image processing, ML/inference, filesystem/network, or platform-specific;
- locate the Android project, target application or library module, package namespace, minimum SDK, build variants, and existing dependency conventions;
- inspect existing Kotlin/Java implementations and public APIs before adding parallel or conflicting code;
- locate representative input fixtures and Python-generated expected outputs;
- check repository instructions, test commands, formatting conventions, and any migration or API documentation.

When the Python project has multiple implementations, identify which one is authoritative for the requested behavior. Do not infer semantics from names alone; trace the data flow and inspect tests or call sites.

### 2. Write the migration contract

Before translating substantial code, define the boundary between Android and the algorithm. Record:

- public Kotlin function or class API;
- input and output types, nullability, ownership, and lifecycle;
- tensor/image layout such as `NCHW` versus `NHWC`, RGB versus BGR, and row stride assumptions;
- numeric precision, rounding, clipping, interpolation, normalization, thresholding, and tolerance expectations;
- coordinate origin, inclusive/exclusive bounds, resize policy, padding, and orientation handling;
- model files, labels, configuration, native libraries, and runtime dependencies;
- threading, cancellation, memory limits, and whether the operation may run on the main thread;
- behavior for empty, malformed, undersized, oversized, or unsupported inputs.

If the contract is ambiguous and different choices would change output behavior or public API, stop before implementing that portion and ask the user for the missing decision.

### 3. Choose Android representations and dependencies

Select representations based on the actual data flow rather than performing mechanical type substitution. Common choices include:

- scalar values → Kotlin primitive types with explicit conversion points;
- Python lists/tuples → typed Kotlin collections or data classes;
- dense numeric arrays → `FloatArray`, `IntArray`, `ByteArray`, a typed tensor wrapper, or an image/matrix type;
- images → `Bitmap`, `Image`, `ByteBuffer`, or OpenCV `Mat`, with explicit ownership and color/channel rules;
- dictionaries → typed Kotlin data classes for stable schemas, or maps only when keys are genuinely dynamic;
- generators/iterators → sequences or explicit loops, with attention to allocation and one-shot behavior;
- NumPy/OpenCV operations → Android-compatible libraries or small local implementations when the operation is simple and dependency cost is justified;
- ONNX or other model inference → the Android runtime version and tensor APIs already supported by the target project.

Prefer the target repository's existing libraries and versions. Do not add a new dependency merely because it resembles the Python package. Confirm Android support, ABI implications, license compatibility, and runtime size before introducing a native or large numerical dependency.

### 4. Implement in small parity-preserving slices

Migrate in dependency order, usually from data conversion and preprocessing through core computation, inference, postprocessing, and public result mapping. Keep each slice reviewable and testable.

For every nontrivial operation, preserve or explicitly review:

- dtype conversion and overflow behavior;
- broadcasting and reduction axes;
- integer division versus floating-point division;
- NaN, infinity, and missing-value behavior;
- array indexing, negative indices, slicing, and boundary conditions;
- sort stability, tie handling, deduplication, and ordering;
- interpolation and resize behavior;
- image channel order, alpha handling, rotation, and coordinate transforms;
- random seeds, nondeterminism, and model/runtime settings;
- resource loading and path resolution on Android.

Use named helpers and data classes for semantic transformations. Avoid hiding important conversions in broad extension functions or implicit casts. Keep platform adapters separate from algorithm code when possible so the core can be tested without a device.

### 5. Validate against the Python reference

Build a parity harness around shared fixtures. Start with small deterministic cases that isolate one operation, then add representative real inputs and edge cases.

Compare at the appropriate level:

- exact equality for labels, counts, shapes, indices, and discrete decisions when determinism is expected;
- absolute and relative tolerances for floating-point values;
- per-pixel or per-element tolerances for images, masks, heatmaps, and tensors;
- structural equality for nested result fields, including ordering and absent-versus-empty semantics;
- resource/model checksums or version identifiers where artifacts must match.

When outputs differ, localize the first divergent stage rather than loosening tolerances immediately. Check layout, dtype, color order, resize/interpolation, rounding, coordinate conventions, model input preparation, and postprocessing before changing the algorithm.

Record known non-equivalences with their cause, impact, expected tolerance, and whether they are accepted or unresolved. If the Python implementation is nondeterministic, compare distributions or invariant properties and document the method.

### 6. Validate on Android

Run the narrowest relevant checks:

- Kotlin compilation for the affected source set and variant;
- unit tests for pure algorithm and conversion logic;
- parity tests using fixtures accessible to the Android test environment;
- instrumented or sample tests for Bitmap, assets, native libraries, model loading, threading, and lifecycle behavior when applicable;
- lint, static analysis, or formatting checks required by the repository.

Measure or inspect memory and runtime behavior for representative inputs. Watch for accidental full-image copies, boxed numeric collections, large temporary tensors, main-thread work, and unreleased native resources.

If an emulator or physical device is unavailable, report the skipped runtime checks. Distinguish JVM/unit validation, emulator validation, and physical-device validation, especially when native code or ABI-specific dependencies are involved.

### 7. Update the integration surface

Update public API documentation, result field descriptions, dependency notes, required permissions, asset/model installation rules, and known limitations when the migration changes host-app behavior. Keep internal migration notes separate from host-app integration instructions.

If the target is an Android Library and the user requests a distributable artifact, hand the completed source to the AAR packaging workflow. Report migration parity and AAR packaging status separately.

## Common mapping guidance

Read the relevant reference before making a nontrivial mapping:

- [language-and-type-mapping.md](references/language-and-type-mapping.md) for Python/Kotlin types, control flow, mutability, and error behavior;
- [numpy-opencv-android.md](references/numpy-opencv-android.md) for arrays, images, channel order, shape, indexing, and common numerical/image operations;
- [onnx-runtime-android.md](references/onnx-runtime-android.md) for model input/output preparation and Android runtime considerations;
- [parity-testing.md](references/parity-testing.md) for fixture design, tolerances, divergence diagnosis, and reporting.

Only read references relevant to the current migration; do not assume every project uses NumPy, OpenCV, or ONNX.

## Required handoff report

Report separately:

- source entry points and target Kotlin files/modules;
- public API and data-contract changes;
- dependencies added, reused, or intentionally not ported;
- Python/Kotlin parity fixtures, comparison method, tolerances, and results;
- unit, instrumented, emulator, and physical-device checks passed, failed, or skipped;
- performance or memory considerations;
- known semantic differences, unresolved risks, and required host-app changes;
- whether a separate AAR build was requested and, if so, its artifact status.

Do not describe a source translation as complete when only compilation has been checked and behavioral parity remains untested.
