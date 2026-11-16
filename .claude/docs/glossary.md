# Glossary

Image data:

- Buffer: image file bytes in memory. All public functions take and return buffers. (`[]byte`)
- Image type: the format of a buffer, detected from magic bytes. (`ImageType`, `vipsImageType`)
- Interpretation: the colour space of an image, for example sRGB or CMYK. (`Interpretation`)
- ICC profile: colour profile that can be removed or converted on save. (`NoProfile`, `InputICC`, `OutputICC`)

Transform:

- Options: the full description of one transform. An empty `Options` value is valid. (`Options`)
- Force: resize to the exact width and height and ignore the aspect ratio. (`Options.Force`)
- Embed: put the resized image on a canvas of the exact size. (`Options.Embed`)
- Extend: the way libvips fills new pixels when it enlarges the canvas. (`Extend`)
- Gravity: which part of the image stays after a crop. `GravitySmart` picks the area with the most interest. (`Gravity`)
- Shrink: integer reduction step before the final resize. (`calculateShrink`)
- Residual: the scale factor that remains after the shrink. (`calculateResidual`)
- Shrink-on-load: the JPEG or WebP decoder scales the image while it decodes, so a large image uses less memory. (`shrinkOnLoad`)
- Autorotate only: private mode that applies only the EXIF orientation. (`Options.autoRotateOnly`, `Image.AutoRotate`)

libvips:

- Bridge: a C function in `vips.h` that wraps a variadic libvips call. (`vips_*_bridge`)
- Operation cache: libvips cache of operations. bimg limits it to 100 MiB and 500 entries by default. (`VipsCacheSetMaxMem`, `VipsCacheSetMax`)
