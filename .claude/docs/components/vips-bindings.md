# libvips bindings

CGO layer between Go and libvips. `vips.h` holds C bridge functions. `vips.go` holds the Go wrappers, the libvips start code and the cache settings.

- Packages: `bimg` (`vips.go`, `vips.h`)
- Used by: [resize-pipeline.md](resize-pipeline.md), [metadata.md](metadata.md), [image-types.md](image-types.md)

## Flow
1. `init` calls `Initialize`.
2. `Initialize` panics if the libvips version is below 7.40. It then locks the OS thread, calls `vips_init`, and sets the cache limits (100 MiB, 500 operations).
3. If `VIPS_CONCURRENCY` is empty, it sets concurrency to 1. If `VIPS_TRACE` is not empty, it enables cache tracing.
4. Each operation calls a bridge function and returns `catchVipsError()` when the C return code is not 0. `catchVipsError` reads and clears the libvips error buffer and calls `vips_thread_shutdown`.
5. `vipsSave` runs `vipsPreSave` (profile removal, colour space, ICC transform) and then calls the encoder for the output type. JPEG is the default encoder.

## Rules
- Free every `C.CString` with `C.free`. Reason: the Go garbage collector does not free C memory.
- Unref each libvips image exactly once. Reason: a second unref frees an image that another reference still uses. `vipsSave` skips the second unref when `vipsPreSave` returns the same pointer.
- `Shutdown` is a no-op when libvips is not initialized. Reason: `initialized` guards `vips_shutdown`.

## Decisions
- Concurrency 1 by default. Reason (from the code comment): a higher libvips concurrency "may generate thread-unsafe issues". The comment links a libvips issue.
- Version checks are in `vips.h` with `#if`. Examples from the file: `vips_find_trim` needs libvips 8.6, smart crop needs 8.5, the JXL, AVIF and HEIF code uses 8.8 to 8.12 branches. Older versions get a fallback or a no-op.
- AVIF loading and saving use the HEIF bridge (`C.HEIF`). Reason: libvips loads AVIF through libheif.

## To add a libvips operation
1. Add a bridge function to `vips.h`. Keep the C signature simple (ints and doubles).
2. Add a Go wrapper in `vips.go` that unrefs its input and returns `catchVipsError()` on failure.
3. Free every `C.CString` that you create.
