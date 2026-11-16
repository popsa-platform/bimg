# Image API

The public entry points. `Image` wraps one buffer and offers a method per common task. Each method builds `Options` and calls `Process`.

- Packages: `bimg` (`image.go`, `resize.go`, `resize_legacy.go`, `file.go`)
- Used by: every use case in [use-cases/](../use-cases/README.md)

## Flow
1. A method such as `Image.ResizeAndCrop` builds an `Options` value.
2. `Image.Process` calls `Resize(i.buffer, o)`.
3. `Resize` calls `resizer` (see [resize-pipeline.md](resize-pipeline.md)).
4. `Image.Process` replaces `i.buffer` with the result and returns it.

## Rules
- `Image` is stateful. After a successful `Process`, the next call works on the new buffer. Reason: callers can chain operations without a copy by hand.
- `Resize` calls `runtime.KeepAlive(buf)` in `resize.go`. Reason: the C code reads the Go buffer, and the garbage collector must not free it early.
- `Image.Extract` sets `Top = -1` when both `top` and `left` are 0. Reason: the pipeline treats `Top == 0 && Left == 0` as "no extract area" (see the last `case` in `extractOrEmbedImage`), and `vipsExtract` clamps negative values to 0.
- `Image.Thumbnail` sets quality 95 and crops to a square. Other methods use the default quality 75.

## Decisions
- `resize_legacy.go` exists for Go below 1.7, and its build tag is `!go1.7`. The module needs Go 1.25.2, so the file never builds. Reason for keeping it: it comes from upstream.
- `Read` and `Write` in `file.go` are helpers that wrap `os.ReadFile` and `os.WriteFile` (mode 0644).

## To add a new Image method
1. Add the method in `image.go`. Build `Options` and return `i.Process(options)`.
2. If the method needs a new option, follow [resize-pipeline.md](resize-pipeline.md#to-add-a-new-option).
3. Add a test in `image_test.go`.
