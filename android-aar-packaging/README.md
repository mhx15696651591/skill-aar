# Android AAR Packaging Skill

A reusable Codex skill for building, inspecting, verifying, archiving, and handing off Android Library AAR artifacts from Gradle-based Android projects.

It is designed to work across Android SDK/library repositories without assuming a specific module name, project path, ABI set, native dependency, model, or resource layout.

## What it does

The skill guides an agent through:

- discovering the actual Gradle project and Android Library modules;
- selecting the requested release, debug, flavor, or ABI-specific artifact;
- keeping formal device builds separate from emulator-oriented test builds;
- running relevant library tests and compilation checks;
- inspecting the exact AAR archive and its embedded resources;
- verifying native libraries against the project's configured ABIs;
- calculating and recording the AAR SHA-256 checksum;
- copying verified artifacts into a new immutable handoff directory;
- reporting build, archive, sample, emulator, and physical-device verification separately.

## Scope

This skill targets Android Library projects that use Gradle and produce `.aar` files for host-app integration.

It does not automatically translate Python code into Kotlin, modify algorithms, publish to Maven, upload files, install applications on devices, or replace existing release artifacts unless the user explicitly requests those actions. Python-to-Kotlin migration is a separate workflow and should be handled by a dedicated skill before using this packaging skill.

## Package structure

```text
android-aar-packaging/
├── SKILL.md
├── README.md
└── agents/
    └── openai.yaml
```

`SKILL.md` contains the agent instructions. `agents/openai.yaml` provides optional UI metadata for Codex-compatible environments.

## Installation

Copy the `android-aar-packaging` directory into the skills directory used by your Codex environment. A typical user-level location is:

```text
<CODEX_HOME>/skills/android-aar-packaging/
```

The exact skills directory depends on the host application and its configuration.

## Usage

Invoke the skill explicitly when needed:

```text
Use $android-aar-packaging to build and verify the release AAR for this Android library.
```

It may also be selected automatically when the task clearly involves packaging or validating an Android Library AAR.

Before building, provide any project-specific requirements that are not discoverable from the repository, such as:

- the intended library module or variant;
- the formal device ABI set;
- a required archive or release directory;
- whether a Sample APK should also be built;
- required host-app documentation;
- whether device or emulator runtime validation is available.

## Artifact status model

The skill keeps these results distinct:

| Status | Meaning |
| --- | --- |
| Built | The selected Gradle task completed and produced an artifact. |
| Inspected | The exact AAR archive, entries, resources, native libraries, and checksum were checked. |
| Sample compiled | A sample or demo APK was built successfully. |
| Emulator verified | The artifact was exercised on an emulator with the stated ABI and configuration. |
| Device verified | The formal device path was exercised on a physical device with the stated ABI and configuration. |

One status must not be presented as proof of another. In particular, an APK is not a substitute for an AAR, and a successful emulator build does not prove behavior of a different native device ABI.

## Project-specific behavior

When the target repository contains a packaging manual, release checklist, validation script, or handoff template, the skill uses those project-specific instructions after discovering the project. Repository conventions take precedence over the example task shapes in `SKILL.md`.

## Validation

The skill package should contain valid frontmatter and no unfinished scaffold placeholders. In a Codex development environment, validate it with the `skill-creator` quick validator when available.

## License

No license file is included yet. Add the license required by your organization or repository policy before publishing, such as MIT or Apache-2.0.
