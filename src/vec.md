# Vec

A `Vec`, short for vector, is a contiguous growable array type, written as
`Vec<T>`. Although we haven't covered user-defined types yet, `Vec` is
implemented using a `struct`.[^1] It supports many familiar operations like
`push`, `pop`, and `len`, but also includes some operations you might not be
familiar with.

## Storing our Lines in a Vector

Let's populate a vector with the input we've been using so far in our rustle
program. There are several ways to create a vector, and the [documentation]
provides plenty of examples. We'll explore a few of them.

### Using `push`

Someone with a C++ background might be inclined to achieve this using a variant
of the following form:

> Hidden code
>
> To make reading the code easier, only the new parts are visible. You can view
> previously seen code by clicking the _Show hidden lines_ button when hovering
> over the code block.

```rust
# fn main() {
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
    let mut lines = Vec::new(); // each call to push mutates the vector
    for line in poem.lines() {
        lines.push(line);
    }

    // format text for debugging purposes
    println!("{lines:?}");
# }
```

We haven't discussed traits yet, so for now, just know that `:?` specifies
formatting the output with the `Debug`[^2] trait. The vector type implements the
`fmt::Debug` trait.

### Using an Iterator

While using `push` is functionally correct, it introduces a mutable variable
unnecessarily. An idiomatic solution would look like this:

```rust
# use std::iter::FromIterator; // this line addresses a rust playground bug
#
# fn main() {
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
    // convert poem into lines
    let lines = Vec::from_iter(poem.lines());
#
#     // format text for debugging purposes
#     println!("{lines:?}");
# }
```

> Favoring immutability
>
> The concept of being _immutable by default_ is one of the many ways Rust
> encourages you to write code that leverages its safety features and supports
> easy concurrency.

## Tracking all Matches

Now that we have a vector containing all the lines in our poem, let's create
another vector to hold the line numbers where the pattern was found.

```rust
# use std::iter::FromIterator; // this line addresses a rust playground bug
#
# fn main() {
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
    let pattern = "all";

#     // convert the poem into lines
#     let lines = Vec::from_iter(poem.lines());
#
    // store the 0-based line number for any matched line
    let match_lines: Vec<_> = lines // inferred type (_)
        .iter()
        .enumerate()
        .filter_map(|(i, line)| match line.contains(pattern) {
            true => Some(i),
            false => None,
        })
        .collect(); // turns anything iterable into a collection
#
#     // format text for debugging purposes
#     println!("{match_lines:?}");
# }
```

Instead of using `from_iter`, you can also use `collect`.

> The inferred type (`_`) in `Vec<_>` _asks_ the compiler to infer the type if
> possible based on the surrounding context.[^2]

## Creating Intervals from Matched Lines

The `map` function invokes the `closure` once for each value in the vector,
passing `line_no` (the line number) to the function. We use this to identify the
lines before and after the match that we want to print. To handle potential
out-of-bounds indexing, we use `saturating_add` and `saturating_sub`.

```rust
# use std::iter::FromIterator; // this line addresses a rust playground bug
#
# fn main() {
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
#     let pattern = "all";
    let before_context = 1;
    let after_context = 1;

#     // convert the poem into lines
#     let lines = Vec::from_iter(poem.lines());
#
#     // store the 0-based line number for any matched line
#     let match_lines: Vec<_> = lines
#         .iter()
#         .enumerate()
#         .filter_map(|(i, line)| match line.contains(pattern) {
#             true => Some(i),
#             false => None,
#         })
#         .collect(); // turns anything iterable into a collection
#
    // create intervals of the form [a,b] with the before/after context
    let intervals: Vec<_> = match_lines
        .iter()
        .map(|line_no| {
            (
                // prevent underflow
                line_no.saturating_sub(before_context),
                // prevent overflow
                line_no.saturating_add(after_context),
            )
        })
        .collect();
#
#     // format text for debugging purposes
#     println!("{intervals:?}");
# }
```

## Merging Intervals

Merging overlapping intervals is straightforward because they are already
sorted, and the ending value of the next interval must be greater than the
current one. We could store the merged intervals in a new Vector, but that would
be inefficient. Instead, we'll make our `intervals` vector mutable and take
advantage of the `dedup_by` iterator adaptor to perform the merge.

```rust
# use std::iter::FromIterator; // this line addresses a rust playground bug
#
# fn main() {
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
#     let pattern = "all";
#     let before_context = 1;
#     let after_context = 1;
#
#     // convert the poem into lines
#     let lines = Vec::from_iter(poem.lines());
#
#     // store the 0-based line number for any matched line
#     let match_lines: Vec<_> = lines
#         .iter()
#         .enumerate()
#         .filter_map(|(i, line)| match line.contains(pattern) {
#             true => Some(i),
#             false => None,
#         })
#         .collect(); // turns anything iterable into a collection
#
#     // create intervals of the form [a,b] with the before/after context
    let mut intervals: Vec<_> = match_lines
#         .iter()
#         .map(|line| {
#             (
#                 line.saturating_sub(before_context),
#                 line.saturating_add(after_context),
#             )
#         })
#         .collect();
#
    // merge overlapping intervals
    intervals.dedup_by(|next, prev| {
        if prev.1 < next.0 {
            false
        } else {
            prev.1 = next.1;
            true
        }
    });
#
#     // format text for debugging purposes
#     println!("{intervals:?}");
# }
```

The [`dedup_by`] iterator adaptor removes consecutive identical elements based
on a provided closure. It starts by calling the closure with the first (`prev`)
and second (`next`) elements in the collection. If the closure returns `false`,
`prev` becomes `next`, `next` is advanced, and the closure is called with the
new elements. If the closure returns `true`, only `next` is advanced. This
behavior allows us to update `prev` to be the merged interval, enabling
efficient merging of overlapping intervals.

> Tuple fields are accessed using a 0-based index, just like array elements.

## Print the Results

With our intervals merged, we can now print out the results correctly!

```rust
# use std::iter::FromIterator; // this line addresses a rust playground bug
#
# fn main() {
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
#     let pattern = "all";
#     let before_context = 1;
#     let after_context = 1;
#
#     // convert the poem into lines
#     let lines = Vec::from_iter(poem.lines());
#
#     // store the 0-based line number for any matched line
#     let match_lines: Vec<_> = lines
#         .iter()
#         .enumerate()
#         .filter_map(|(i, line)| match line.contains(pattern) {
#             true => Some(i),
#             false => None,
#         })
#         .collect(); // turns anything iterable into a collection
#
#     // create intervals of the form [a,b] with the before/after context
#     let mut intervals: Vec<_> = match_lines
#         .iter()
#         .map(|line| {
#             (
#                 line.saturating_sub(before_context),
#                 line.saturating_add(after_context),
#             )
#         })
#         .collect();
#
#     // merge overlapping intervals
#     intervals.dedup_by(|next, prev| {
#         if prev.1 < next.0 {
#             false
#         } else {
#             prev.1 = next.1;
#             true
#         }
#     });
#
    // print the lines
    for (start, end) in intervals {
        for (line_no, line) in lines
                .iter()
                .enumerate()
                .take(end + 1)
                .skip(start) {
            println!("{}: {}", line_no + 1, line)
        }
    }
# }
```

## We Did It!

We have a working rustle program! Take some time to review everything we've
covered so far. The rest of the course is going to move quickly.

# Summary

We covered quite a bit in this section:

- `collect` converts any iterable into a collection.
- `saturating_add` and `saturating_sub` prevent overflow.
- `skip` creates an iterator that skips elements until `n` elements are skipped
  or the end of the iterator is reached, whichever comes first.
- `take` creates an iterator that yields elements until `n` elements are yielded
  or the end of the iterator is reached, whichever comes first.
- The `println!` macro supports the [`:?`] format specifier, which is intended
  to help with debugging Rust code. All public types should implement the
  `fmt::Debug trait`.[^3]

# Next

Let's dive into the concepts of ownership and borrowing.

[^1]: Refer to the [`Vec` source code] see its full implementation. The `struct`
    definition is on line [397].

[^2]: The [Inferred type] is part of the type system in Rust.

[^3]: Details about the `Debug` trait can be found
    [here](https://doc.rust-lang.org/std/fmt/trait.Debug.html).

[397]: https://doc.rust-lang.org/src/alloc/vec/mod.rs.html#397
[documentation]: https://doc.rust-lang.org/std/vec/struct.Vec.html
[inferred type]: https://doc.rust-lang.org/reference/types/inferred.html?highlight=Vec%3C_%3E#inferred-type
[`:?`]: https://doc.rust-lang.org/std/fmt/index.html#fmtdisplay-vs-fmtdebug
[`dedup_by`]: https://docs.rs/itertools/latest/itertools/trait.Itertools.html#method.dedup_by
[`vec` source code]: https://doc.rust-lang.org/src/alloc/vec/mod.rs.html
