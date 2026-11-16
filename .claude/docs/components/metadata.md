# Metadata

Reads size, channels, alpha, profile, colour space and EXIF fields from a buffer, without a transform.

- Packages: `bimg` (`metadata.go`)
- Used by: [use-cases/read-image-metadata.md](../use-cases/read-image-metadata.md)

## Flow
1. `Metadata` loads the buffer with `vipsRead`.
2. It reads `Xsize`, `Ysize` and `Bands` from the libvips image.
3. It reads each EXIF tag with `vipsExifStringTag` or `vipsExifIntTag`. A missing tag gives an empty string or 0.
4. It unrefs the image and returns `ImageMetadata`.

## Rules
- `vipsExifShort` cuts the text after the first ` (`. Reason: libvips returns values such as `Canon (Make)`, and callers want the plain value.
- EXIF tag names are the constants at the top of `metadata.go`. Add a field in three places: the constant, the `EXIF` struct and the `Metadata` function.

## To add an EXIF field
1. Add the tag constant (`exif-ifdN-<Name>`).
2. Add the field to `EXIF`. Use `string` or `int`, the same as the `vipsExif*Tag` function that you call.
3. Fill it in `Metadata`.
4. Update the expected value in `metadata_test.go`.
