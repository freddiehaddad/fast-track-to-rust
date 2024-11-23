We already know that the `collect` method converts anything iterable into a
collection. The first time we used `collect`, we specified the type next to the
identifier. Here, we're specifying the type next to the `collect` method. This
is called _turbofish_[^3] and is commonly used. Neither version is wrong; the
choice is usually made based on readability or which option requires fewer edits
if changes are needed.

> Type inference
>
> When using the collect method to convert an iterator into a collection, the
> compiler might not know what type of collection you want. In these situation,
> you must specify the type.

# Next

With the ability to open and read files in place, we're now ready to add support
for command line arguments!

[Non-operator Symbols]:
  https://doc.rust-lang.org/book/appendix-02-operators.html#non-operator-symbols

---

[^1]:
    Strings are implemented as `Vec<u8>` in Rust. Reference the [API] for
    details.

[^3]:
    _Turbofish_ notation is mentioned in _Table B-4: Generics_ in the Rust
    documentation for [Non-operator Symbols].
