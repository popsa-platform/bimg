# A program resizes or crops an image

## Who and when
- A Go program that has an image buffer. It calls `bimg.NewImage(buf)` and a method such as `Resize`, `ResizeAndCrop`, `Thumbnail` or `Process(Options{...})`.

## Steps
1. Image API: build `Options` and call `Process`. See [image-api.md](../components/image-api.md).
2. Resize pipeline: load, rotate by EXIF, scale, crop or embed, apply effects, save. See [resize-pipeline.md](../components/resize-pipeline.md).
3. libvips bindings: each step calls libvips. See [vips-bindings.md](../components/vips-bindings.md).
4. The program reads the new buffer from the return value or from `Image.Image()`.

## Data
- Read: the input buffer, and ICC profile files if `InputICC` or `OutputICC` is set.
- Written: nothing. The result is a new buffer in memory.

## If a step fails
- Empty buffer: `Image buffer is empty`.
- Unknown format: `Unsupported image format`.
- Output type that libvips cannot save: `Unsupported image output type`.
- Extract or smart crop larger than `MaxSize()`: `Maximum image size exceeded`.
- libvips failure: an error with the libvips message. The `Image` keeps its old buffer.

## Config and tools
- `Options`: all transform settings.
- `VIPS_CONCURRENCY`, `VIPS_TRACE`: see the README.
- `SetMaxsize`: changes the size limit.

## Rules
- Set `Enlarge` to allow output larger than the input. Reason: without it, an image smaller than the target keeps its size.
- Pass `Type` to change the format. Reason: the default output type is the input type.
