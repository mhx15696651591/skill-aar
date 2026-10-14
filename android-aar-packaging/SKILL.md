---
name: android-aar-packaging
description: Build, inspect, verify, archive, and hand off Android Library AAR artifacts from a Gradle project. Use when packaging an Android SDK/library for host-app integration, selecting ABIs, validating native and embedded resources, or producing a reproducible release handoff; do not use for ordinary Android app feature work that does not produce an AAR.
metadata:
  short-description: Build and verify Android Library AARs
---

# Android AAR Packaging

Use this skill to turn an Android library project into a verified, clearly identified AAR handoff. Adapt to the repository's actual Gradle structure, build conventions, ABI configuration, native dependencies, embedded resources, and release policy. Do not assume module names, paths, model files, native libraries, or a particular Android toolchain.

## Operating principles

- Inspect the repository before choosing commands. Identify the project root, Gradle wrapper, Android library module(s), application/sample modules, build variants, product flavors, ABI controls, signing requirements, native dependencies, and embedded assets.
- Treat the AAR as a library artifact for host-app integration. An APK is a separate test/demo artifact and must never be reported as a substitute for the AAR.
- Preserve unrelated user changes. Do not delete build directories, `.cxx` directories, caches, or existing release archives as routine cleanup.
- Distinguish the artifact being built: formal release, debug/internal package, flavor-specific package, ABI-specific test package, or sample APK. State the choice before building when ambiguity could change the deliverable.
- Do not let an emulator-oriented ABI build overwrite or get confused with the formal device release. If the project emits the same filename for multiple ABI configurations, copy each result to an explicitly named temporary location and verify it before proceeding.
- Compilation proves that the selected source compiles; it does not prove native runtime behavior, algorithm parity, device compatibility, or host-app integration.

## Workflow

### 1. Discover the project

Inspect, at minimum:

- repository status and existing user changes;
- `gradlew`/`gradlew.bat`, `settings.gradle(.kts)`, root build files, and `gradle.properties`;
- library modules and their `com.android.library` configuration;
- release/debug variants and flavors;
- `abiFilters`, external native build configuration, prefab/CMake/NDK dependencies, and any ABI or feature properties;
- `src/*/assets`, model/resource directories, JNI libraries, generated resources, and runtime dependency declarations;
- existing packaging scripts, release notes, output folders, and project-specific packaging documentation.

Use the repository's configured JDK, Android Gradle Plugin, Gradle wrapper, NDK, SDK, and dependency caches. Do not silently switch toolchains to make a build pass. If the required toolchain is unavailable, report the exact limitation.

If the source project has a separate algorithm or reference implementation, compare only the affected modules and data flow when behavior changed. Pay particular attention to public result fields, masks/filters, coordinate systems, asset versions, and native/runtime versions. Record meaningful differences; do not claim pixel-level equivalence from a successful Kotlin compilation.

### 2. Select the build target

Choose the smallest relevant task that produces the requested artifact. Common patterns are illustrative, not mandatory:

```text
<wrapper> :<library-module>:assemble<Variant>
<wrapper> :<library-module>:bundle<Variant>
<wrapper> :<library-module>:test
<wrapper> :<library-module>:compile<Variant>Kotlin
<wrapper> :<sample-module>:assemble<Variant>
```

For a formal device AAR:

- use the project's release variant and its default production ABI/resource configuration;
- do not add emulator-only ABI flags unless the user explicitly requested an emulator artifact;
- ensure the chosen variant does not depend on debug-only source sets or test-only files.

For an emulator or ABI-specific test AAR:

- label it as a test artifact before building;
- pass only the project-supported ABI/configuration properties;
- keep it separate from the formal release output;
- do not infer device-native behavior from an emulator build, especially when native dependencies differ by ABI.

For an APK, build it only when sample behavior or API integration is in scope, and report it separately from the AAR.

### 3. Build and test

Run the relevant library unit tests and release compilation, or the release assemble task that includes compilation. Add sample APK construction for API or behavior changes when practical. Run reference-language tests only when the correct environment and dependencies are available.

Do not install to a device, upload artifacts, publish to a Maven repository, replace a separately named deliverable, or modify external systems unless the user explicitly requested that side effect.

If a build fails, classify the failure before changing configuration: source/compiler error, dependency/toolchain issue, ABI/native-link issue, missing asset/resource, test failure, or environment limitation. Make the narrowest in-scope correction and preserve the original failure context.

### 4. Verify the exact AAR

Verify the file produced by the selected task, not merely a similarly named older file:

- confirm path, modification time, nonzero size, and successful task result;
- inspect the archive as a ZIP/JAR and confirm `classes.jar`, manifest/resources, and all expected embedded assets;
- for native libraries, confirm the expected ABI directories and `.so` files, based on the current Gradle/native configuration;
- confirm important runtime dependencies are packaged, declared, or intentionally supplied by the host app according to the project's design;
- check duplicate or stale model/resource versions when the library embeds data files;
- compute SHA-256 for the exact AAR and retain it in the handoff record.

Do not require native libraries, models, or a particular archive entry when the project does not use them. Verify against discovered configuration rather than a fixed checklist.

### 5. Archive and hand off

When the user requests a distributable or release handoff, copy the verified AAR to a new immutable, clearly named archive directory outside the transient build output. A useful convention is `<archive-root>/<date>_<version-or-variant>`, but follow an existing repository convention when one exists.

Before copying, check that the destination does not already contain a handoff. Do not overwrite historical artifacts; choose a new versioned directory. Recompute the copied file's SHA-256 and confirm it matches the canonical output.

Include only documentation required for host-app integration, such as API/field specifications, dependency requirements, ABI support, permissions, initialization, and known limitations. Keep internal demo operation guides separate unless the user asks for them.

## Required handoff report

Report the following separately and explicitly:

- source project and library module;
- selected variant/flavor and ABI/native configuration;
- canonical AAR path and immutable handoff path, if created;
- file size, timestamp, and SHA-256;
- verified archive entries, native ABIs, assets, and relevant dependencies;
- tests and compilation checks passed, failed, or skipped, with reasons;
- whether a Sample APK was built and whether it was actually run;
- whether emulator behavior and physical-device behavior were verified;
- source/reference alignment status and known algorithm or runtime limitations;
- any action that remains for the host-app integrator.

Do not collapse “built,” “archive inspected,” “sample compiled,” “emulator ran,” and “formal device path ran” into one generic success statement.

## Project-specific extensions

If the repository contains a packaging manual, release checklist, validation script, or required handoff template, read and follow it after discovery. Project-specific instructions override the example commands in this skill, but not the safety rules about preserving changes, separating artifacts, and avoiding archive overwrites.
