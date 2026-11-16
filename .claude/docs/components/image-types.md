# Image types

Detects the format of a buffer and tells which formats the installed libvips can load and save.

- Packages: `bimg` (`type.go`, `vipsImageType` and `readImageType` in `vips.go`)
- Used by: [resize-pipeline.md](resize-pipeline.md), [metadata.md](metadata.md)

## Flow
1. `vipsImageType` returns `UNKNOWN` for buffers shorter than 12 bytes.
2. It compares magic bytes in this order: JPEG, GIF, PNG, TIFF, PDF, WebP, SVG, ImageMagick, HEIF variants, AVIF, JXL.
3. Types that are optional in libvips (all except JPEG and PNG) are checked with `IsTypeSupported` first.
4. `IsImageTypeSupportedByVips` fills the `SupportedImageTypes` map on first use (lazy) and answers from the map after that.

## Rules
- `SupportedImageTypes` is filled lazily. Reason (from the code comment): libvips cannot answer at package bootstrap. A `sync.RWMutex` guards the map.
- SVG detection uses `IsSVGImage`: the buffer must be text (`isBinary` checks the first 24 bytes), and a regular expression must match after HTML comments are removed.
- HEIF is detected by the `ftyp` brands `heic`, `mif1`, `msf1`, `heis` and `hevc`. AVIF is the brand `avif`.
