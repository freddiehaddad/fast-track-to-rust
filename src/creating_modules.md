# Creating Modules

In the previous section, we defined an `enum` and `struct`. However, we didn't
actually create a module! In this section, we will. Before diving into the
details, the Rust documentation provides a [Module Cheat Sheet] which will be a
handy reference on your Rust journey.

Since we're building our module to work in this online book, we'll deviate
slightly from standard practice. Typically, modules are created in separate
files. For instance, our `Interval` would be placed in a new file called
`interval.rs`.

```console
rustle
+ Cargo.lock
+ Cargo.toml
+ src
  + interval.rs
  + main.rs
```

> Document generation
>
> As we develop our `interval` module, it's a great opportunity to learn about
> [`rustdoc`] and documentation generation. The `interval` module will be
> annotated to enable documentation generation. If you're building the code
> locally, the simplest way to generate and view documentation is by using
> `cargo doc --open`.

Let's get started!

[module cheat sheet]: https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html#modules-cheat-sheet
[`rustdoc`]: https://doc.rust-lang.org/rustdoc/index.html
