# Flutter Foundation Models — Agent Instructions

Pub package: a Flutter/Dart plugin that mirrors Apple's Foundation Models
framework on iOS 26+ / macOS 26+. Three sibling packages in a Melos
workspace: main plugin, annotations, code generator.

## Public API parity is the product
- The Dart API exists to mirror Swift's Foundation Models API. **If Swift
  has a symbol or shape, Dart should match it as closely as Dart allows.**
  See the table in `README.md` (Swift → Dart mapping). Do not invent
  alternative names or "more Dart-y" wrappers without explicit permission.
- Any change to a public class, method, parameter name, or generated
  code shape in `flutter_foundation_models_annotations` or
  `flutter_foundation_models_gen` is a **breaking change for pub users**.
  Treat the public API as frozen unless the user explicitly asks to break it.

## Scope discipline
- Only modify code/files directly related to the asked task. No drive-by
  refactors, renames, or "improvements" to neighboring code in any of the
  three packages.
- If you notice something worth fixing elsewhere, mention it at the end.
  Don't touch it unless asked.
- Before changing the `@Generable` codegen output shape, the platform
  channel method names/payloads, or the package dependency graph in
  `melos.yaml`, stop and describe what's about to change. Wait for
  confirmation.

## Simplest solution first
- Implement the simplest thing that works. No new abstraction layers,
  factories, or "flexibility" that wasn't requested.
- Do not add dartdoc comments, type annotations, or extra validation to
  code you didn't change. (Existing dartdoc is part of the API contract —
  don't strip it either.)

## Uncertainty and assumptions
- Flag uncertainty before answering. "I'm not sure" beats a confident guess
  — especially for Swift API behavior and `build_runner` codegen quirks.
- If a fact can be verified by reading the Swift FM docs or the existing
  generated output, read it. Don't infer.
- When intent is ambiguous, ask before writing code.

## Protected surfaces (do not modify without explicit permission)
- Public API surface of all three packages — anything reachable via
  `export` in `lib/` is a contract.
- Platform channel name + method signatures between Dart and the
  iOS/macOS implementations.
- Generated code shape from `flutter_foundation_models_gen` — downstream
  packages depend on the `$<Name>Generable` extension surface.
- `melos.yaml` and inter-package version constraints — bumping a version
  changes what pub resolves for users.

## Build / publish
- `melos bootstrap` after pulling deps.
- `dart run build_runner build` in any package that needs codegen
  regenerated.
- NEVER run `dart pub publish` (or `flutter pub publish`) unless the user
  explicitly asks to publish in the *current* message. One explicit ask =
  one publish. pub.dev publishes are irreversible.
- `git push`, `git push --force`, `git reset --hard`, branch deletion,
  `rm -rf` — confirm in the current message before running.

## Change summary
After any non-trivial edit, end with:
- **Files changed:** `<list>`
- **Files intentionally not touched:** `<list, if relevant>`
- **Follow-up needed:** `<API surface impact, codegen impact, or test gaps>`
