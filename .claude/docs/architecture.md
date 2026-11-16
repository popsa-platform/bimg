# Architecture

`bimg` is a Go library for Go programs that need to transform images or read image metadata. It keeps the Go API small and lets libvips (C) do the pixel work.

Terms: [glossary.md](glossary.md). Use cases: [use-cases/](use-cases/README.md).

## Components
- Image API: the `Image` type and package functions that callers use. [components/image-api.md](components/image-api.md)
  - `image.go`, `resize.go`, `resize_legacy.go`, `file.go`
- Resize pipeline: turns `Options` into an ordered set of libvips steps. [components/resize-pipeline.md](components/resize-pipeline.md)
  - `resizer.go`, `options.go`
- libvips bindings: CGO wrappers, C bridge functions, libvips start and cache settings. [components/vips-bindings.md](components/vips-bindings.md)
  - `vips.go`, `vips.h`
- Image types: format detection and the list of formats that libvips supports. [components/image-types.md](components/image-types.md)
  - `type.go`, `vipsImageType` in `vips.go`
- Metadata: size, colour data and EXIF fields. [components/metadata.md](components/metadata.md)
  - `metadata.go`

## Consumers
- Go services in the Popsa platform that import `github.com/popsa-platform/bimg`: they pass image buffers and get new buffers. The module has no `require` entries, so consumers only add the libvips system dependency. This repo does not list the consumers.

## External connections
- libvips (C library, found by `pkg-config`)
  - When: at package init (`init` calls `Initialize`) and on every call.
  - Why: decode, transform and encode the pixels.
  - If it fails: `Initialize` panics with `unable to start vips!` if `vips_init` fails. Operation errors return as Go `error` values with the libvips error text (`catchVipsError`).

## Design patterns
- Facade: `Image` and `Resize` hide `Options` and the libvips calls. Reason: callers do one call per task.
- Pipeline: `resizer` runs fixed steps in a fixed order, and each step is skipped if its options are empty. Reason: one code path for all operations, so the combination of operations stays predictable.
- Thin bridge: each libvips call has a C function in `vips.h` and one Go wrapper in `vips.go`. Reason: the variadic libvips C API cannot be called from Go.
