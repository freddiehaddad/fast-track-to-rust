# Iterators

Iterators are pervasive in Rust, and delving into them in full detail would
necessitate renaming this course to **Slow Track to Rust**. We'll cover the
basics and direct you to the documentation for all the intricate details about
[iterators].

The [Control Flow](./control_flow.md) section explained that the call to
`lines()` in the `for` loop returns an iterator. We say that the `for` loop
_consumes_ the iterator. However, in Rust, we can do much more than just consume
iterators!

## Iterator Adaptors

New iterators can be created using [iterator adaptors]. These adaptors generate
new iterators that modify some aspect of the original. Let's use one to print
line numbers.

The `enumerate` function creates an iterator that yields the current iteration
count along with the value from the previous iterator, as tuples[^1] in the form
`(index, value)`. In the loop, the type for `i` is `usize`, and the type for
`line` is `&str`.

```rust
# fn main() {
#     let pattern = "him";
#     let poem = "I have a little shadow that goes in and out with me,
#                 And what can be the use of him is more than I can see.
#                 He is very, very like me from the heels up to the head;
#                 And I see him jump before me, when I jump into my bed.
#
#                 The funniest thing about him is the way he likes to grow -
#                 Not at all like proper children, which is always very slow;
#                 For he sometimes shoots up taller like an india-rubber ball,
#                 And he sometimes gets so little that there's none of him at all.";
#
    for (i, line) in poem.lines().enumerate() {
        if line.contains(pattern) {
            println!("{}: {line}", i + 1);
        }
    }
# }
```

## Advanced Iterators

We can take it a step further by using the `filter_map` iterator adapter to
return an iterator that includes only the items we want!

```rust
# fn main() {
#     let pattern = "him";
#     let poem = "I have a little shadow that goes in and out with me,
#                 And what can be the use of him is more than I can see.
#                 He is very, very like me from the heels up to the head;
#                 And I see him jump before me, when I jump into my bed.
#
#                 The funniest thing about him is the way he likes to grow -
#                 Not at all like proper children, which is always very slow;
#                 For he sometimes shoots up taller like an india-rubber ball,
#                 And he sometimes gets so little that there's none of him at all.";
#
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
# }
```

Whoa! What just happened? We're on the fast track to learning Rust, so we're
picking up the pace! Let's break this down because this code snippet needs some
unpacking.

[^1]: A [tuple] is a collection of values of different types and is constructed
    using parentheses `()`.

[iterator adaptors]: https://doc.rust-lang.org/book/ch13-02-iterators.html?search=#methods-that-produce-other-iterators
[iterators]: https://doc.rust-lang.org/book/ch13-02-iterators.html#processing-a-series-of-items-with-iterators
[tuple]: https://doc.rust-lang.org/rust-by-example/primitives/tuples.html#tuples
