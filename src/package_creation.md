# Package Creation

Let's make sure all the prerequisites are in place. You should have followed the
[installation](installation.md) instructions to prepare your development
environment. After those steps are complete, you should be able to run the
following commands:

```console
$ rustc --version
rustc 1.81.0 (eeb90cda1 2024-09-04)

$ cargo --version
cargo 1.81.0 (2dbb1af80 2024-08-20)
```

> The version numbers might be different, but the output should look relatively
> similar.

If the above commands worked, you're ready to go!

1. Use `cargo new rustle` to create a new Rust package named `rustle` for our
   project:

   ```console
   $ cargo new rustle
       Creating binary (application) `rustle` package
   note: see more `Cargo.toml` keys and their definitions at
   https://doc.rust-lang.org/cargo/reference/manifest.html
   ```

1. Navigate to the `rustle` directory and use `cargo run` to build and run the
   program:

   ```console
   $ cd rustle
   $ cargo run
      Compiling rustle v0.1.0 (S:\projects\git\rustle)
       Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.43s
        Running `target\debug\rustle.exe`
   Hello, world!
   ```

1. Explore some of the other actions you can perform with `cargo` using
   `cargo --help`.

   - `cargo build` - compile the current package
   - `cargo build --release` - build the project in release mode (with
     optimizations)
   - `cargo check` - analyze the current package and report errors (no object
     files generated)
   - `cargo test` - run unit tests
   - and more...

# Summary

In this section, we:

- Created a package.
- Compiled and ran the boilerplate code.
- Learned a bit about Cargo.

# Next

Let's see what `cargo new` actually did!
