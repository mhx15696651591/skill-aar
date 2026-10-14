# Android SDK Skills

一组面向 Android SDK 开发的 Codex skills，覆盖从 Python 算法迁移到 Kotlin/Android，再到 Android Library AAR 构建、验证和交付的完整流程。

This repository contains reusable Codex skills for Android SDK development. The current workflow is split into two independent skills:

```text
Python algorithm
      ↓
python-to-kotlin-android-porting
      ↓
Kotlin/Android implementation and parity validation
      ↓
android-aar-packaging
      ↓
Verified AAR handoff
```

## Skills

### [`python-to-kotlin-android-porting`](./python-to-kotlin-android-porting)

用于将 Python 算法、图像处理流程、预处理逻辑或推理流程迁移到 Kotlin/Android。

主要关注：

- Python 入口、依赖和数据契约分析；
- Python 类型、NumPy、OpenCV、张量和图像数据到 Kotlin/Android 的映射；
- Android 端的资源、模型、线程、内存和生命周期处理；
- Python 与 Kotlin 的数值、像素、结构化结果一致性验证；
- 迁移过程中的已知差异、风险和宿主 APP 接口变更。

这个 skill 把 Python 实现视为行为参考，而不是简单进行逐行语法翻译。它不会承诺任意 Python 代码都能自动转换；遇到服务端依赖、训练代码、动态反射或 Android 不支持的科学计算依赖时，会要求重新设计或明确记录差异。

### [`android-aar-packaging`](./android-aar-packaging)

用于构建、检查、验证、归档和交付 Android Library AAR。

主要关注：

- 自动发现 Gradle 项目、Library module、Sample module 和构建 Variant；
- 区分正式设备 AAR、模拟器测试 AAR、Debug 包和 Sample APK；
- 按工程实际配置检查 ABI、JNI native library、模型和资源；
- 执行相关编译和测试；
- 检查 AAR 内容并计算 SHA-256；
- 将已验证的 AAR 归档到不可覆盖的版本化目录；
- 分开报告构建、归档检查、Sample、模拟器和真机验证结果。

这个 skill 不负责 Python 转 Kotlin，也不会默认发布 Maven、上传制品、安装 APK 或覆盖历史交付物。

## Recommended workflow

当一个 Python 算法需要成为 Android SDK 的一部分时，建议按以下顺序使用：

1. 使用 `python-to-kotlin-android-porting` 分析 Python 实现和 Android 目标工程。
2. 明确输入输出、数据类型、坐标系、通道顺序、阈值、精度和资源依赖。
3. 将算法分阶段迁移到 Kotlin，并用 Python 输出作为 reference 做 parity testing。
4. 在 Android 环境中执行 Kotlin 编译、单元测试和必要的运行时测试。
5. 使用 `android-aar-packaging` 构建目标 Variant 的 AAR。
6. 检查 AAR 内的 classes、资源、模型、JNI 库和 ABI 内容。
7. 记录文件大小、时间、SHA-256、测试状态和已知限制后交付。

两个 skill 可以配合使用，但彼此不强依赖：

- 只需要迁移算法时，可以单独使用 `python-to-kotlin-android-porting`；
- 已经有 Kotlin Android Library、只需要打包时，可以单独使用 `android-aar-packaging`。

## Usage in Codex

将对应 skill 目录安装到 Codex 的 skills 目录后，可以显式调用：

```text
Use $python-to-kotlin-android-porting to port this Python algorithm to the Android library and validate behavioral parity.
```

```text
Use $android-aar-packaging to build, verify, and hand off the release AAR for this Android library.
```

也可以在请求中补充项目特定信息，例如：

- Python 入口文件或函数；
- Android 目标工程和 Library module；
- 目标公开 API；
- 代表性输入、golden output 和容差；
- 最低 Android SDK 和目标 ABI；
- 是否需要构建 Sample APK；
- AAR 归档目录和宿主 APP 集成文档要求。

## Repository structure

```text
skill-aar/
├── README.md
├── android-aar-packaging/
│   ├── SKILL.md
│   ├── README.md
│   └── agents/
│       └── openai.yaml
└── python-to-kotlin-android-porting/
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

## Design principles

- 不将某个业务、项目路径、模块名、模型名或 ABI 集合写死在通用 skill 中。
- 不把“编译通过”当成算法一致性或设备运行验证。
- 不把 Sample APK 当成 AAR，也不把模拟器结果当成其他 ABI 的真机结果。
- 不覆盖历史交付物，正式制品需要有明确路径、版本标识和校验值。
- 对无法直接映射的 Python 语义、依赖或运行时行为明确记录，而不是静默近似替换。
- 优先遵循目标仓库已有的构建脚本、版本、测试和交付规范。

## License

本仓库暂未指定许可证。公开发布前，请根据个人或组织要求补充 `LICENSE` 文件，例如 MIT 或 Apache-2.0。
