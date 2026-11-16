# A program reads image metadata

## Who and when
- A Go program that needs size, orientation or EXIF data. It calls `bimg.Metadata(buf)`, `bimg.Size(buf)` or `Image.Metadata()`.

## Steps
1. Metadata: load the buffer and read the fields. See [metadata.md](../components/metadata.md).
2. Image types: detect the format of the buffer. See [image-types.md](../components/image-types.md).

## Data
- Read: the input buffer.
- Written: nothing.

## If a step fails
- Unknown or unsupported format: `Unsupported image format`.
- Missing EXIF tag: the field has an empty string or 0, with no error.
