# protofuzzer

Fuzzing experiment focused on **malformed Protocol Buffers wire messages** — feeding arbitrary or intentionally broken binary payloads into the protobuf parser to look for crashes, memory safety issues, and unexpected behavior.

## Goal

Exercise the C++ binary decode path (`ParseFromArray` / parse context) with inputs that are not valid protobufs: truncated varints, illegal wire types, overlong length fields, mismatched groups, invalid UTF-8 in strings, and similar edge cases. Valid parse failures are expected; sanitizer findings (ASan / UBSan) or crashes are the interesting outcomes.

## Layout

| Path | Role |
|------|------|
| [`protobuf/`](protobuf/) | Shallow clone of [protocolbuffers/protobuf](https://github.com/protocolbuffers/protobuf) (`main`). Source under test; not built yet. |
| *(planned)* `fuzz/` | Out-of-tree libFuzzer harness and seed corpus (not added yet). |

Relevant upstream pieces for this experiment:

- `protobuf/src/google/protobuf/message_lite.cc` — `ParseFromArray`
- `protobuf/src/google/protobuf/parse_context.h` — parse / recursion context
- `protobuf/src/google/protobuf/test_messages_proto2.proto` / `test_messages_proto3.proto` — schemas that cover the full wire surface (used by upstream fuzz / conformance tests)

## Status

- [x] Clone protobuf source (no configure / build)
- [ ] Build instrumented `libprotobuf` + `protoc` (ASan, UBSan, fuzzer)
- [ ] Add `LLVMFuzzerTestOneInput` harness for proto2 / proto3 `TestAllTypes`
- [ ] Seed corpus of malformed wire blobs and a short smoke run

## Approach (when harness lands)

1. Compile protobuf with Clang sanitizers and `fuzzer-no-link`.
2. Generate C++ from the two `test_messages_*.proto` files.
3. Fuzz both message types with raw bytes; on successful parse, serialize and parse again.
4. Keep seeds small; drop oversized inputs so single cases cannot stall the fuzzer.

## Notes

- The `protobuf/` tree is a nested git clone; treat it as vendored upstream for local fuzzing, not as something to rewrite in place.
- Submodules (Abseil, etc.) were left uninitialized until a build step is needed.
