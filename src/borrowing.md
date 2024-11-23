# Borrowing

Given the error in the previous section about borrowing a value after it has
been moved, let's now focus on how to _borrow_ a value.

## References

Recall from the section on the [string slice `str`] that we said it's usually
seen in it's _borrowed_ form `&str`. The `&` operator[^1] in the prefix position
represents a borrow.

## Borrowing `lines`

In `create_intervals`, we _borrow_ `lines` by prefixing the parameter's type
with an `&`, changing it to `&Vec<&str>`, and by prefixing the variable `lines`
in `main` to `&lines`. After making these changes and re-running the program, we
can see that it now works.

```rust
# use std::iter::FromIterator; // this line addresses a rust playground bug
#
fn create_intervals(
#     before_context: usize,
#     after_context: usize,
#     match_lines: Vec<usize>,
    lines: &Vec<&str>,
#) -> Vec<(usize, usize)> {
#     match_lines
#         .iter()
#         .map(|line| {
#             (
#                 line.saturating_sub(before_context),
#                 line.saturating_add(after_context),
#             )
#         })
#         .collect()
# }

fn main() {
#     let poem = "I have a little shadow that goes in and out with me,
#                 And what can be the use of him is more than I can see.
#                 He is very, very like me from the heels up to the head;
#                 And I see him jump before me, when I jump into my bed.
#
#                 The funniest thing about him is the way he likes to grow -
#                 Not at all like proper children, which is always very slow;
#                 For he sometimes shoots up taller like an india-rubber ball,
#                 And he sometimes gets so little that there’s none of him at all.";
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
    // create intervals of the form [a,b] with the before/after context
    let mut intervals =
        create_intervals(before_context, after_context, match_lines, &lines);
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
#     // print the lines
#     for (start, end) in intervals {
#         for (line_no, line) in
#             lines.iter().enumerate().take(end + 1).skip(start)
#         {
#             println!("{}: {}", line_no + 1, line)
#         }
#     }
}
```

> Moving vs Borrowing
>
> - Why is it possible to _move_ `match_lines` without causing an error?
> - Considering heap allocations, what advantages might there be in _moving_
>   `match_lines` instead of _borrowing_ it?

# Summary

Although the concepts of ownership and borrowing are relatively straightforward,
they can be frustrating when learning Rust. Reading through the official
documentation on [Understanding Ownership] will certainly help overcome this
challenge. Keep the following points in mind as you continue on your journey
with Rust:

**Ownership Rules**

- Each value in Rust has an owner.
- There can only be one owner at a time.
- When the owner goes out of scope, the value is dropped.

**Borrowing Rules**

- At any given time, you can have either one mutable reference or any number of
  immutable references.
- References must always be valid.

[string slice `str`]: types.md#string-slice-str
[borrowed pointer type]:
  https://doc.rust-lang.org/book/appendix-02-operators.html#operators
[Understanding Ownership]:
  https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html

# Next

With our new understanding of ownership and borrowing, let's switch our focus to
error handling.

---

[^1]:
    The `&` operator can have different meanings depending on the context. For
    example, when used an infix operator, it becomes a bitwise AND.
