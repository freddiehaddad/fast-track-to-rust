# Error Handling

Many programs use mechanisms like exceptions for handling errors, but Rust takes
a different approach. Rust doesn't have exceptions. Instead, it uses the
[`Result`] type for recoverable errors and the `panic!` macro for unrecoverable
errors. Similar to the [`Option`] `enum` we previously discussed, [`Result`] is
another `enum`.

## `Result<T, E>`

`Result<T, E>` is the type used for returning and propagating errors. Similar to
`Option`, it has two variants: `Ok(T)`, which represents success and contains a
value, and `Err(E)`, which represents an error and contains an error value. The
`T` and `E` are generic type parameters that we'll discuss in more detail soon.
For now, it's important to know that `T` represents the type of the value
returned in a success case within the `Ok` variant, and `E` represents the type
of the error returned in a failure case within the `Err` variant.

```rust,noplayground
enum Result<T, E> {
   Ok(T),
   Err(E),
}
```

## Additional Resources

Several sections in the Rust documentation thoroughly cover [error handling].

Here are some useful links:

- [Unrecoverable Errors with panic!]: Useful information on how to troubleshoot
  code that panics.
- [Recoverable Errors with Result]: Deep dive into `Result` and general error
  handling.
- [To panic! or Not to panic!]: General guidelines on how to decide whether to
  panic in library code.
- [The question mark operator]: Useful for propagating errors to the calling
  function.[^1]

[^1]: We'll cover the question mark operator `?` when we refactor the code.

[error handling]: https://doc.rust-lang.org/book/ch09-00-error-handling.html
[recoverable errors with result]: https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html
[the question mark operator]: https://doc.rust-lang.org/reference/expressions/operator-expr.html#the-question-mark-operator
[to panic! or not to panic!]: https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html
[unrecoverable errors with panic!]: https://doc.rust-lang.org/book/ch09-01-unrecoverable-errors-with-panic.html
[`option`]: https://doc.rust-lang.org/std/option/enum.Option.html
[`result`]: https://doc.rust-lang.org/std/result/enum.Result.html
