# Borrowing

Given the error in the previous section about borrowing a value after it has
been moved, let's now focus on how to _borrow_ a value.

## References

Recall from the section on the [string slice `str`] that we said it's usually
seen in it's _borrowed_ form `&str`. The `&` operator[^1] in the prefix position
represents a borrow. In `find_matching_lines`, `pattern` is _borrowed_.

```rust,noplayground
fn find_matching_lines(lines: Vec<&str>, pattern: &str) -> Vec<usize>
```

## Borrowing `lines`

In `find_matching_lines`, we can _borrow_ `lines` by prefixing the parameter's
type with an `&`, changing it to `&Vec<&str>`, and by prefixing the variable
`lines` in `main` to `&lines`. After making these changes and re-running the
program, we can see that it now works.

```rust
# use std::iter::FromIterator; // this line addresses a rust playground bug
#
fn find_matching_lines(lines: &Vec<&str>, pattern: &str) -> Vec<usize> {
    lines
        .iter()
        .enumerate()
        .filter_map(|(i, line)| match line.contains(pattern) {
            true => Some(i),
            false => None,
        })
        .collect() // turns anything iterable into a collection
}

fn main() {
    let poem = "I have a little shadow that goes in and out with me,
                And what can be the use of him is more than I can see.
                He is very, very like me from the heels up to the head;
                And I see him jump before me, when I jump into my bed.

                The funniest thing about him is the way he likes to grow -
                Not at all like proper children, which is always very slow;
                For he sometimes shoots up taller like an india-rubber ball,
                And he sometimes gets so little that there's none of him at all.";

    // command line arguments
    let pattern = "all";
    let before_context = 1;
    let after_context = 1;

    // convert the poem into lines
    let lines = Vec::from_iter(poem.lines());

    // store the 0-based line number for any matched line
    let match_lines = find_matching_lines(&lines, pattern);

    // create intervals of the form [a,b] with the before/after context
    let mut intervals: Vec<_> = match_lines
        .iter()
        .map(|line| {
            (
                line.saturating_sub(before_context),
                line.saturating_add(after_context),
            )
        })
        .collect();

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

## Continuing to Refactor

Let's create another function to handle the creation of our intervals. Here is
the function signature:

```rust,noplayground
fn create_intervals(
    lines: Vec<usize>,
    before_context: usize,
    after_context: usize,
) -> Vec<(usize, usize)>
```

This is the code that we'll transfer into the new function:

```rust,noplayground
// create intervals of the form [a,b] with the before/after context
let mut intervals: Vec<_> = match_lines
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
# use std::iter::FromIterator; // this line addresses a rust playground bug
#
fn find_matching_lines(lines: &Vec<&str>, pattern: &str) -> Vec<usize> {
    lines
        .iter()
        .enumerate()
        .filter_map(|(i, line)| match line.contains(pattern) {
            true => Some(i),
            false => None,
        })
        .collect() // turns anything iterable into a collection
}

fn create_intervals(
    lines: Vec<usize>,
    before_context: usize,
    after_context: usize,
) -> Vec<(usize, usize)> {
    lines
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
                And he sometimes gets so little that there's none of him at all.";

    // command line arguments
    let pattern = "all";
    let before_context = 1;
    let after_context = 1;

    // convert the poem into lines
    let lines = Vec::from_iter(poem.lines());

    // store the 0-based line number for any matched line
    let match_lines = find_matching_lines(&lines, pattern);

    // create intervals of the form [a,b] with the before/after context
    let mut intervals =
        create_intervals(match_lines, before_context, after_context);

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

> Moving vs Borrowing
>
> - Why is it possible to _move_ `match_lines` without causing an error?
> - Considering heap allocations, what advantages might there be in moving
>   `match_lines` instead of _borrowing_ it?

# Exercise

Develop a function named merge_intervals and transfer the specified code from
main into this function, making any necessary updates. Construct another
function called print_results and relocate the specified code from main into
this function, updating it as needed.

- Create a function named `merge_intervals` and move the specified code from
  `main` into this function, making any necessary updates.
  ```rust,noplayground
  // merge overlapping intervals
  intervals.dedup_by(|next, prev| {
      if prev.1 < next.0 {
          false
      } else {
          prev.1 = next.1;
          true
      }
  });
  ```
- Create another function called `print_results` and move the specified code
  from `main` into this function, updating it as needed.
  ```rust,noplayground
  // print the lines
  for (start, end) in intervals {
      for (line_no, line) in
          lines.iter().enumerate().take(end + 1).skip(start)
      {
          println!("{}: {}", line_no + 1, line)
      }
  }
  ```
- Modify `main` to utilize these newly created functions.

> You can complete these exercises by updating the most recent version of the
> code provided above.

<details>
<summary>Solution</summary>

```rust
use std::iter::FromIterator; // this line addresses a rust playground bug

fn find_matching_lines(lines: &Vec<&str>, pattern: &str) -> Vec<usize> {
    lines
        .iter()
        .enumerate()
        .filter_map(|(i, line)| match line.contains(pattern) {
            true => Some(i),
            false => None,
        })
        .collect() // turns anything iterable into a collection
}

fn create_intervals(
    lines: Vec<usize>,
    before_context: usize,
    after_context: usize,
) -> Vec<(usize, usize)> {
    lines
        .iter()
        .map(|line| {
            (
                line.saturating_sub(before_context),
                line.saturating_add(after_context),
            )
        })
        .collect()
}

fn merge_intervals(intervals: &mut Vec<(usize, usize)>) {
    // merge overlapping intervals
    intervals.dedup_by(|next, prev| {
        if prev.1 < next.0 {
            false
        } else {
            prev.1 = next.1;
            true
        }
    })
}

fn print_results(intervals: Vec<(usize, usize)>, lines: Vec<&str>) {
    for (start, end) in intervals {
        for (line_no, line) in
            lines.iter().enumerate().take(end + 1).skip(start)
        {
            println!("{}: {}", line_no + 1, line)
        }
    }
}

fn main() {
    let poem = "I have a little shadow that goes in and out with me,
                And what can be the use of him is more than I can see.
                He is very, very like me from the heels up to the head;
                And I see him jump before me, when I jump into my bed.

                The funniest thing about him is the way he likes to grow -
                Not at all like proper children, which is always very slow;
                For he sometimes shoots up taller like an india-rubber ball,
                And he sometimes gets so little that there's none of him at all.";

    // command line arguments
    let pattern = "all";
    let before_context = 1;
    let after_context = 1;

    // convert the poem into lines
    let lines = Vec::from_iter(poem.lines());

    // store the 0-based line number for any matched line
    let match_lines = find_matching_lines(&lines, pattern);

    // create intervals of the form [a,b] with the before/after context
    let mut intervals =
        create_intervals(match_lines, before_context, after_context);

    // merge overlapping intervals
    merge_intervals(&mut intervals);

    // print the lines
    print_results(intervals, lines);
}
```

</details>

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

# Next

With our new understanding of ownership and borrowing, let's switch our focus to
error handling.

[^1]: The `&` operator can have different meanings depending on the context. For
    example, when used an infix operator, it becomes a bitwise AND.

[string slice `str`]: types.md#string-slice-str
[understanding ownership]: https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html
