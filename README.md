# [Little CMS](http://www.littlecms.com) wrapper for [Rust](https://www.rust-lang.org/)

Convert and apply color profiles with a safe abstraction layer for the LCMS library.

See [API reference](https://docs.rs/lcms2/) for Rust functions and the [LCMS2 documentation HTML](https://kornelski.github.io/rust-lcms2-sys/)/[PDF](http://www.littlecms.com/LittleCMS2.8%20API.pdf) for more background information about the functions.

```rust
use lcms2::*;

fn example() -> Result<(), std::io::Error> {
    let icc_file = include_bytes!("custom_profile.icc"); // You can use Profile::new_file("path"), too
    let custom_profile = Profile::new_icc(icc_file)?;

    let srgb_profile = Profile::new_srgb();

    let t = Transform::new(&custom_profile, PixelFormat::RGB_8, &srgb_profile, PixelFormat::RGB_8, Intent::Perceptual);

    // Pixel struct must have layout compatible with PixelFormat specified in new()
    let source_pixels: &[rgb::RGB<u8>] = …;
    t.transform_pixels(source_pixels, destination_pixels);

    // If input and output pixel formats are the same, you can overwrite them instead of copying
    t.transform_in_place(source_and_dest_pixels);

    Ok(())
}
```

To apply an ICC profile from a JPEG:

```rust
if b"ICC_PROFILE\0" == &app2_marker_data[0..12] {
   let icc = &app2_marker_data[14..]; // Lazy assumption that the profile is smaller than 64KB
   let profile = Profile::new_icc(icc)?;
   let t = Transform::new(&profile, PixelFormat::RGB_8,
       &Profile::new_srgb(), PixelFormat::RGB_8, Intent::Perceptual);
   t.transform_in_place(&mut rgb);
}
```

There's more in the `examples` directory.

This crate requires Rust 1.33 or later. It's up to date with LCMS 2.9, and should work with 2.6 to 2.9.

## Threads

In LCMS all functions are in 2 flavors: global and `*THR()` functions. In this crate this is represented by having functions with `GlobalContext` and `ThreadContext`. Create profiles, transforms, etc. using `*_context()` constructors to give them their private coontext, which makes them sendable between threads (i.e. they're `Send`).

By default `Transform` does not implement `Sync`, because LCMS2 has a thread-unsafe cache in the transform. You can set `Flags::NO_CACHE` to make it safe (this is checked at compile time).
