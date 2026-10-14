# Python to Kotlin Android Porting Skill

A reusable Codex skill for migrating Python algorithms, preprocessing pipelines, and inference flows into Kotlin code for Android applications or libraries.

The skill treats the Python implementation as a behavioral reference. It focuses on preserving data semantics, numerical and image-processing behavior, Android runtime constraints, and evidence of Python/Kotlin parity.

## What it does

- discovers Python entry points, dependencies, data contracts, and reference behavior;
- identifies the Android target module, API surface, build variants, and existing dependencies;
- maps Python types and scientific-computing operations to Kotlin and Android representations;
- ports code in small, testable slices instead of promising blind line-by-line translation;
- handles common NumPy, OpenCV, tensor, image, and model-inference concerns when those dependencies are present;
- creates parity fixtures and compares Python and Kotlin results with appropriate tolerances;
- checks Android-specific memory, threading, asset, model, lifecycle, and ABI concerns;
- documents semantic differences, skipped checks, and host-app integration changes.

## Scope and limitations

This skill is intentionally project-agnostic. It does not assume a particular business domain, Python package, Android module, model, image format, or public API.

It does not guarantee automatic conversion of arbitrary Python code. Dynamic typing, reflection, server-only services, multiprocessing, training pipelines, unsupported SciPy operations, and dependencies without Android support may require redesign or a native implementation. The skill should identify those risks instead of silently replacing behavior with an approximation.

It does not automatically build or publish an AAR. Once the Kotlin implementation is ready, use a separate Android AAR packaging workflow when a distributable library artifact is required.

## Package structure

```text
python-to-kotlin-android-porting/
├── SKILL.md
├── README.md
├── agents/
│   └── openai.yaml
└── references/
    ├── language-and-type-mapping.md
    ├── numpy-opencv-android.md
    ├── onnx-runtime-android.md
    └── parity-testing.md
```

## Installation

Copy the `python-to-kotlin-android-porting` directory into the skills directory used by your Codex environment. A typical user-level location is:

```text
<CODEX_HOME>/skills/python-to-kotlin-android-porting/
```

The exact skills directory depends on the host application and its configuration.

## Usage

Invoke it explicitly when a Python implementation needs to move to Android/Kotlin:

```text
Use $python-to-kotlin-android-porting to port this Python inference pipeline to the Android library module and validate Kotlin results against the Python reference.
```

Useful context to provide includes:

- the Python entry point or requested module;
- the target Android project and module;
- the intended public API;
- representative input fixtures and expected outputs;
- required numerical or image tolerances;
- Android minimum SDK and supported ABIs;
- whether the final result should be packaged as an AAR.

## Validation model

The skill separates these claims:

| Status | Meaning |
| --- | --- |
| Ported | Kotlin implementation exists for the requested scope. |
| Compiles | The affected Kotlin source set compiles. |
| Unit tested | Local deterministic tests pass. |
| Parity checked | Python and Kotlin outputs were compared with stated fixtures and tolerances. |
| Android runtime checked | The implementation was exercised in its intended Android environment. |
| AAR packaged | A separate packaging workflow produced and inspected the requested AAR. |

Compilation is not parity, and parity is not physical-device validation.

## License

No license file is included yet. Add the license required by your organization or repository policy before publishing, such as MIT or Apache-2.0.
