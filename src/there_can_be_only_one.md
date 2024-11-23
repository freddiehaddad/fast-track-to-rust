# There Can Be Only One

Currently, our entire grep program is contained within the `main` function. This
approach was chosen to address the challenges of teaching a course in a way that
introduces important concepts logically and in an easily digestible manner.
However, the time has come to refactor some of the code from the main function
into separate functions.

Let's start by creating a function to handle the creation of our intervals. Here
is the function signature:

```rust,noplayground
fn create_intervals(
    before_context: usize,
    after_context: usize,
    match_lines: Vec<usize>,
    lines: Vec<&str>,
) -> Vec<(usize, usize)>
```

This is the code that we'll transfer into the new function:

```rust,noplayground
// create intervals of the form [a,b] with the before/after context
let intervals: Vec<_> = match_lines
    .iter()
    .map(|line| {
        (
            line.saturating_sub(before_context),
            line.saturating_add(after_context),
        )
    })
    .collect();
```

Here is the revised code with the changes implemented. Review it and run the
code.

```rust,editable
use std::iter::FromIterator; // this line addresses a rust playground bug

fn create_intervals(
    before_context: usize,
    after_context: usize,
    match_lines: Vec<usize>,
    lines: Vec<&str>,
) -> Vec<(usize, usize)> {
    match_lines
        .iter()
        .map(|line| {
            (
                line.saturating_sub(before_context),
                line.saturating_add(after_context),
            )
        })
        .collect()
}

fn main() {
    let poem = "I have a little shadow that goes in and out with me,
                And what can be the use of him is more than I can see.
                He is very, very like me from the heels up to the head;
                And I see him jump before me, when I jump into my bed.

                The funniest thing about him is the way he likes to grow -
                Not at all like proper children, which is always very slow;
                For he sometimes shoots up taller like an india-rubber ball,
                And he sometimes gets so little that there’s none of him at all.";

    let pattern = "all";
    let before_context = 1;
    let after_context = 1;

    // convert the poem into lines
    let lines = Vec::from_iter(poem.lines());

    // store the 0-based line number for any matched line
    let match_lines: Vec<_> = lines
        .iter()
        .enumerate()
        .filter_map(|(i, line)| match line.contains(pattern) {
            true => Some(i),
            false => None,
        })
        .collect(); // turns anything iterable into a collection

    // create intervals of the form [a,b] with the before/after context
    let mut intervals =
        create_intervals(before_context, after_context, match_lines, lines);

    // merge overlapping intervals
    intervals.dedup_by(|next, prev| {
        if prev.1 < next.0 {
            false
        } else {
            prev.1 = next.1;
            true
        }
    });

    // print the lines
    for (start, end) in intervals {
        for (line_no, line) in
            lines.iter().enumerate().take(end + 1).skip(start)
        {
            println!("{}: {}", line_no + 1, line)
        }
    }
}
```

## Uh-oh!

A minor code change caused an issue with the program. Fortunately, the Rust
compiler offers helpful information for diagnosing the problem. However, if
you're not familiar with Rust's ownership rules, understanding this error can be
challenging. Let's break down the error and understand what went wrong. Here are
the key details from the error message, cleaned up for readability:

```text
let lines = Vec::from_iter(poem.lines());
    ----- move occurs because `lines` has type `Vec<&str>`, which does not implement
          the `Copy` trait

create_intervals(before_context, after_context, match_lines, lines);
                                                             ----- value moved here

lines.iter().enumerate().take(end + 1).skip(start)
^^^^^^^^^^^^ value borrowed here after move
```

### Unpacking the Error

Copy trait, value moved, value borrowed. What the heck does all this mean?

#### Copy Trait

We'll explore [traits] in more detail later in the course. For now, just know
that when a type implements the [`Copy`] trait, its values are duplicated when
assigned to a new variable. This means that after an assignment, both the
original and the new variable can be used independently.

The `lines` vector we passed to the `create_intervals` function does _not_
implement the [`Copy`] trait.

#### Value Moved

By default, variable bindings in Rust follow _move semantics_.[^1] When a value
is _moved_, its ownership is transferred to the new variable, rendering the
original variable invalid and unusable.

Since the `lines` vector does not implement the [`Copy`] trait, its value was
_moved_, rendering the original value in _main_ invalid.

#### Value Borrowed

Because the `lines` variable in `main` becomes invalid due to the _move_, any
attempt to [_borrow_] or reference its value is invalid. This is why the
compiler generates the message "value borrowed here after move".

> Move Semantics
>
> Rust's move semantics play an integral part in ensuring memory safety by
> detecting common memory-related errors, like null pointer dereferencing,
> buffer overflows, and use-after-free, during compile time, thereby preventing
> them from happening at runtime.

[traits]: https://doc.rust-lang.org/book/ch10-02-traits.html
[`Copy`]: https://doc.rust-lang.org/std/marker/trait.Copy.html
[_borrow_]:
  https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html#references-and-borrowing

---

[^1]:
    There are some exceptions in Rust. For example, most primitive types
    implement the [`Copy`] trait.
