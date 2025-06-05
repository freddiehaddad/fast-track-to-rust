# Closures

In Rust, [closures] are functions that can capture their surrounding
environment, clearly express the programmer's intent, and achieve this with
low-level performance. They are quite common in functional programming
languages.

Closures in Rust have some distinct characteristics:

- Parameters are enclosed with `||` instead of `()`.
- Curly braces `{}` around the function body are optional for single-line
  expressions.

## Returning to our Loop

```rust,editable
fn main() {
    let pattern = "him";
    let poem = "I have a little shadow that goes in and out with me,
                And what can be the use of him is more than I can see.
                He is very, very like me from the heels up to the head;
                And I see him jump before me, when I jump into my bed.

                The funniest thing about him is the way he likes to grow -
                Not at all like proper children, which is always very slow;
                For he sometimes shoots up taller like an india-rubber ball,
                And he sometimes gets so little that there's none of him at all.";

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
}
```

In the code snippet, the closure is:

```rust, noplayground
|(i, line)| match line.contains(pattern) {
    true => Some((i + 1, line)),
    false => None,
}
```

## `filter_map`

The [`filter_map`] [^1] function creates an iterator that both filters and maps.
It calls a closure and filters out the results that are `None`, leaving us with
an iterator of tuples containing the line number and line for all matches of our
pattern. When the `for` loop consumes the iterator, we only need to print the
results, as we are left with only the lines that contained the pattern!

> Recall that `enumerate` returns an iterator of `tuple`s. `filter_map` receives
> the `tuple` as the argument to the closure.

# Exercise

- Replace `filter_map` with `map` and `filter` to achieve the same output.
  Notice how `filter_map` results in more efficient and concise code.

<details>
<summary>Solution</summary>

```rust,editable
fn main() {
    let pattern = "him";
    let poem = "I have a little shadow that goes in and out with me,
                And what can be the use of him is more than I can see.
                He is very, very like me from the heels up to the head;
                And I see him jump before me, when I jump into my bed.

                The funniest thing about him is the way he likes to grow -
                Not at all like proper children, which is always very slow;
                For he sometimes shoots up taller like an india-rubber ball,
                And he sometimes gets so little that there's none of him at all.";

    for (line_no, line) in poem
        .lines()
        .enumerate()
        .map(|(i, line)| (i + 1, line))
        .filter(|(_line_no, line)| line.contains(pattern))
    {
        println!("{line_no}: {line}");
    }
}
```

> The underscore `_` prefix in `_line_no` is how we tell the Rust compiler that
> we are intentionally ignoring the first argument. Without it the compiler will
> complain.

</details>

# Summary

In this section we:

- introduced the `Option` `enum`
- learned about closures
- worked with iterators
- learned about the `mut` keyword

# Next

Onward to collections!

[^1]: Separate `filter` and `map` iterators also exist.

[closures]: https://doc.rust-lang.org/book/ch13-01-closures.html
[`filter_map`]: https://doc.rust-lang.org/std/iter/struct.FilterMap.html
