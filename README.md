# Master thesis files
This repo contains the two repositories that were used for the code in my thesis. 

## FFI-Checker-Driver
Contains various executables and libraries that are required to run MiriPBT. 

### `miripbt_build`
To be added to a `build.rs`  file, it'll make sure to rerun the pbt generator each time a Rust file is changed. 
All it really does is run `cargo pbtgen`, and report the path of the generated file. 

### `miripbt_format`
This crate contains the actual format used by the pbt generator and MiriPBT. It is a limited representation of the Rust soucecode that contains just enough to make everything work. 

### `miripbt_macros` and `miripbt_macros_impl`
These proc macros can be applied to functions in a rust program to mark them for MiriPBT to actually alter the input arguments. 

The actual macros and implementation of them are split up to make development a bit easier since it doesn't break rust-analyzer every time you type a single word. 

### `sample` and `sample_two`
These crates were used to test if MiriPBT worked. Nothing important here except a very extensive format json file if you generate it. **Use this crate to see how to use MiriPBT**

### `pbtgen` (but really the `src` folder)
Contais the actual crate for `cargo-pbtgen`. The `cargo-pbtgen` binary sets up the application to actually run the regular `pbtgen` binary, which is a `rustc_driver`. 

All the driver does is look for any functions annotated with the `miripbt::tool` attribute, and then transform the sigunature to `miripbt_format`. 

### `structure_provider`
The structure provider, uses a http connection and the `miripbt_format` to generate valid arguments for a function. It is supposed to also generate mutability changes, but since that was more difficult than expected it was never fully implemented. 

### Usage
For how to use this crate, check the `to_copy` file, and replace `/root` with the actual path to the `ffi-checker-driver` repo. 

## MIRI
Since the actual MiriLLI repo is difficult to work with (requires LLVM 16 which wasn't shipped anymore), I've done all modifications to Miri itself instead. 

All the code from the commit pointed to in this repo (which is simply the `julius/detect` branch), to the last commit in the `old_miri` branch can be integrated onto the last commit in the `mirilli` repo. 

### Requirements for MiriLLI/MiriPBT
* You need LLVM 16 installed, and available in your path. 
* A lot of patience, I never got building just miri(lli/pbt) to work, so you'll have to recompile everything every time (`./x build` in the `rust` repo). 
* The `config.toml` file in this repo, you need some extra binaries compiled if you want to use `rust-analyzer` for example, once again don't forget to change the path. 
* A small change to the `cargo-miri/Cargo.toml` file, you'll need to forcibly pin the `rustc-build-sysroot` to version `0.5.2` (`rustc-build-sysroot = "=0.5.2"`)

### Actually running MiriPBT
Assuming you got it all compiled, you will need to add a local rustup toolchain to the path via `rustup toolchain link <name> <path>`, something like `rustup toolchain link miripbt ./build/host/stage2` should work if you are in the root of the `rust` repo. 

Once that's done you'll need to either set an override (run `rustup toolchain override miripbt` in the folder of your test project, **not the rust repo**), or use `cargo +miripbt <command>` every time. 

Finally you'll need to set some options via the `MIRIFLAGS` environment variable, most notably the `-Zmiri-pbt-file=` variable should point to the `miripbt_format.json` file for the current project. Remember that if you use the build script it'll tell you what file to point to. 

### Examples
```sh
MIRIFLAGS="-Zmiri-pbt-file=/home/me/thesis-example/target/debug/build/thesis-example-ef9fdaf3545121f3/out/pbtgen/debug/build/thesis-example-ef9fdaf3545121f3/out/miripbt_format_3095207193173024579.json -Zmiri-pbt-stop-first" cargo miri run
```