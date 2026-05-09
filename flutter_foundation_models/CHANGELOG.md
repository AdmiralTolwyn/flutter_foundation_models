## 0.3.1+vacuumbreather.3 (fork)

### Hygiene Pass
- **CI added** — `.github/workflows/ci.yml` runs `dart analyze` on all three
  packages (plugin, annotations, codegen) on every push and PR.
- **`README.md` documents melos bootstrap** — cloning + `flutter pub get`
  inside the example app fails without `melos bootstrap` first because the
  three packages reference each other via path. Now spelled out in the
  top-level README.

## 0.3.1+vacuumbreather.2 (fork)

### Bug Fixes
- **`FlutterTool.swift` force unwraps removed** — `arguments.jsonString.data(using:
  .utf8)!`, `try! JSONSerialization`, and `String(data:encoding:)!` all crashed
  on malformed tool arguments. Replaced with `guard`/`try` that throw
  `PigeonError` with descriptive codes (`INVALID_UTF8`, `ENCODE_ERROR`).
- **`LanguageModelSession.dispose()` stream cleanup race** — `_isDisposed` was
  set *after* cancelling streams and destroying the session. A stream completion
  callback firing during `dispose()` could mutate `_activeStreams` concurrently.
  Now `_isDisposed` is set first and `_activeStreams` is snapshot-cleared before
  cancellation, preventing the race.
- **Silent JSON parse failures logged** — `_parseJson()` in
  `LanguageModelSession` and `_parseJsonString()` in `FlutterApiImpl` caught all
  exceptions and returned `{}`. Parse errors are now logged via `debugPrint` in
  debug builds so data corruption is visible during development.
- **Tool lookup throws `PlatformException`** — `FlutterApiImpl.invokeTool()`
  threw generic `Exception` on missing session/tool, which Pigeon could not map
  back to a structured error on the Swift side. Now throws `PlatformException`
  with codes `SESSION_NOT_FOUND` / `TOOL_NOT_FOUND`.
- **`_unused` placeholder collision-safe** — The auto-injected placeholder for
  zero-property struct schemas used the fixed name `_unused`, which would collide
  if a Dart class had a field with that name. Now uses
  `_placeholder_<UUID-prefix>` to guarantee uniqueness. Also validates that
  non-empty struct schemas have no duplicate property names.

## 0.3.1+vacuumbreather.1 (fork)

Fork-specific patches that fix two issues seen when using tool calling with
streaming text responses.

### Bug Fixes
- **Empty schema crash** — Tools that take no arguments previously threw
  `invalidSchema("no valid properties found in struct")` because Apple's
  `DynamicGenerationSchema` rejects empty struct schemas. The plugin now
  auto-injects a hidden optional `_unused: Bool` placeholder property when the
  Dart side provides an empty `StructGenerationSchema`. No app changes needed.
- **Empty/`"null"` snapshots during tool calls** — While the model is invoking a
  tool, the underlying `streamResponse` emits empty or literal `"null"` content
  snapshots that briefly clear partial UI text. The text streaming path now
  filters these out before forwarding to Flutter, so consumers get a clean
  monotonically-growing text stream.

## 0.3.0

### Breaking Changes
- `respondTo()` now returns `TextResponse` instead of `String`
  - Use `response.content` to get the text
  - Also provides `response.transcriptEntries` for entries created during this response
- `respondToWithSchema()` now returns `StructuredResponse` instead of `GeneratedContent`
  - Use `response.content` to get the generated content
  - Also provides `response.rawContent` and `response.transcriptEntries`

### New Features
- **Response wrappers** - `TextResponse` and `StructuredResponse` provide content plus metadata
- **List generation** - Generate arrays of @Generable types
  - `GenerationSchema.array(schema)` - Create array schema from item schema
  - `content.toList(fromContent)` - Convert array content to typed `List<T>`
  - `content.toPartialList(fromPartialContent)` - For streaming arrays
- **Generation errors** - Typed `GenerationException` matching Swift's `GenerationError`
  - `GenerationErrorType` enum with all Swift error types
  - `debugDescription` for additional debugging info

## 0.2.2

- **Transcript support** - Access conversation history and continue sessions
  - `session.transcript` - Get the current conversation transcript
  - `LanguageModelSession.createWithTranscript()` - Create a session from a previous transcript
  - `Transcript.toJson()` / `Transcript.fromJson()` - Serialize/deserialize transcripts
  - Type-safe transcript entries: `TranscriptPrompt`, `TranscriptResponse`, `TranscriptToolCalls`, `TranscriptToolOutput`, `TranscriptInstructions`

## 0.2.1

- Update documentation

## 0.2.0

### Breaking Changes
- `LanguageModelSession` now uses async factory method `LanguageModelSession.create()` instead of constructor
- Moved `isAvailable` from `LanguageModelSession` to `SystemLanguageModel.isAvailable`

### New Features
- **Swift Package Manager support** - Plugin now supports both SPM and CocoaPods
- **SystemLanguageModel class** - Manage language models with configuration options
  - `SystemLanguageModel.defaultModel` - Access the default system model
  - `SystemLanguageModel.create()` - Create custom model with adapter, useCase, or guardrails
  - `SystemLanguageModel.isAvailable` - Check if Foundation Models API is available
  - `SystemLanguageModel.availability` - Get detailed availability info with unavailability reason
- **Adapter support** - Load custom adapters
  - `Adapter.create(name:)` - Create adapter by name
  - `Adapter.fromAsset(assetPath)` - Create adapter from Flutter asset
- **UseCase configuration** - `UseCase.general` and `UseCase.contentTagging`
- **Guardrails configuration** - `Guardrails.defaultGuardrails` and `Guardrails.permissiveContentTransformations`
- **Text streaming** - `streamResponseTo()` for plain text streaming without schema
- **Session prewarm** - `session.prewarm()` to reduce latency on first request
- **Session state** - `session.isResponding` to check if session is actively generating

## 0.1.1

- Add runtime availability check with `LanguageModelSession.isAvailable()`
- Lower minimum iOS version to 16.0 (API requires iOS 26+ at runtime)
- Improve documentation
- Clean up published package (exclude unnecessary example files)

## 0.1.0

- Initial release
- Text generation with `respondTo()`
- Structured output with `respondToWithSchema()`
- Streaming support with `streamResponseToWithSchema()`
- Tool use support
- Generation options (sampling mode, temperature, max tokens)
- System instructions support
- Availability checking with `LanguageModelSession.isAvailable()`
- Supports iOS 16+ (Foundation Models API requires iOS 26+ at runtime)
