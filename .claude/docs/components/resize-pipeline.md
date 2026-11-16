# Resize pipeline

`resizer` in `resizer.go` runs one transform from `Options`. `options.go` defines `Options` and the enums that it uses.

- Packages: `bimg` (`resizer.go`, `options.go`)
- Used by: [image-api.md](image-api.md)

## Flow
1. `loadImage`: error `Image buffer is empty` for an empty buffer. Else `vipsRead` detects the type and loads the image.
2. `applyDefaults`: `Quality` 75, `Compression` 6, output `Type` = input type, `Interpretation` sRGB. If `Palette` is true, `Speed` becomes 3.
3. Check `IsTypeSupportedSave(o.Type)`. If false, return `Unsupported image output type`.
4. If `autoRotateOnly`: `vipsAutoRotate`, save, return.
5. `rotateAndFlipImage`: EXIF orientation (unless `NoAutoRotate`), then `Rotate`, `Flip`, `Flop`.
6. If the image was rotated and the input was JPEG, HEIF or AVIF, `getImageBuffer` re-encodes the image as JPEG (quality 100, metadata stripped) and uses it as `buf` for shrink-on-load.
7. `normalizeOperation`: if only `Width` or `Height` is set and no mode flag is set, set `Force`.
8. `imageCalculations`, `calculateShrink`, `calculateResidual`: compute the scale. Without `Enlarge` and `Force`, an image that is smaller than the target in both dimensions keeps its size.
9. Shrink-on-load for JPEG, or for WebP on libvips 8.3 or newer, when `shrink >= 2`.
10. `zoomImage`, then `transformImage` (shrink, reduce or affine, then extract, smart crop, embed or trim), then `applyEffects`.
11. Text watermark, image watermark, `imageFlatten`, gamma, brightness, contrast.
12. `saveImage` calls `vipsSave` (see [vips-bindings.md](vips-bindings.md)).

## Rules
- Each `vips*` wrapper takes ownership of its input image and calls `g_object_unref` on it. Reason: libvips images are reference counted, and the pipeline passes each image forward once.
- `Options` fields with the zero value mean "not set". Reason: `applyDefaults` and the `should*` helpers use `== 0` checks. A side effect: `Compression` 0 and `Quality` 0 cannot be requested.
- `Force` clears `Crop` and `Embed` inside `transformImage`. Reason: an exact resize has nothing to crop or embed.
- `SetMaxsize` limit applies to `vipsExtract` and `vipsSmartCrop` only. Reason: only these two wrappers check `maxSize`.

## Decisions
- `Gravity == GravitySmart` and the deprecated `SmartCrop` flag lead to the same branch in `extractOrEmbedImage`.
- The shrink for bicubic interpolation is smaller than the factor (`factor * 3 / windowSize`). Reason (from the code comment): shrink less and affine more for interpolators with a window of 4 pixels or more.

## To add a new option
1. Add the field to `Options` in `options.go`.
2. Add a `vips*` wrapper in `vips.go` and a bridge function in `vips.h` if libvips needs one.
3. Call it from `resizer` at the right step. Skip it when the field has its zero value.
4. If the option changes the saver, add it to `vipsSaveOptions` and to `saveImage`.
5. Add a test in `resizer_test.go`.
