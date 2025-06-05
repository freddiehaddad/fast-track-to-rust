# Option

The `Option` type represents an optional value that can either be `Some` with a
value or `None`. They are prevalent in Rust and serve various purposes. Options
are often used with pattern matching (i.e., `match` expression). The
documentation for [`Option`] provides extensive details on its usage.

`Some(T)` and `None` are two variants of the `Option<T>` [`enum`].[^1]

```rust,noplayground
pub enum Option<T> {
    None,
    Some(T),
}
```

> We haven't discussed [generics] yet. For now, just note that the `T` in the
> `enum` represents any type.

## Returning to our Loop

In the code snippet, we check if the pattern is a substring of the current line.
If it is, we return a `tuple` with the line number (`i+1`)[^2] and the `line`,
enclosed within the `Some` variant of the `Option` `enum`. If not, we return the
`None` variant.

```rust,noplayground
    for (line_no, line) in
        poem.lines()
            .enumerate()
            .filter_map(|(i, line)| match line.contains(pattern) {
                true => Some((i + 1, line)),
                false => None,
            })
    {
        println!("{line_no}: {line}");
    }
```

[^1]: [`enum`]s in Rust are similar to those of other compiled languages like C,
    but have important differences that make them considerably more powerful.

[^2]: Recall that grep uses 1-based line numbering. `enumerate` begins at 0.

[generics]: https://doc.rust-lang.org/rust-by-example/generics.html#generics
[`enum`]: https://doc.rust-lang.org/std/keyword.enum.html
[`option`]: https://doc.rust-lang.org/std/option/
