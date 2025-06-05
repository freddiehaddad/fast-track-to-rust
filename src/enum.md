# Enum

An _[enumeration]_, commonly known as an _[enum]_, is declared using the keyword
`enum`. Compared to other languges, enums in Rust offer greater flexibility and
power. For instance, they support generics (as seen with the `Option` `enum`)
and can encapsulate values within their variants! [^1]

## Using an `enum` for Errors

For our rustle program, we'll create an `enum` to represent possible errors
during `Interval` operations. For example, when a user creates an `Interval`,
the starting value must be less than or equal to the ending value. Additionally,
if a user wants to merge two intervals, they must overlap. If these conditions
aren't met, we'll return an error (`Err`) using one of our `enum` variants.

## Defining an `enum`

Below is the `IntervalError` `enum`, which lists the errors that we may need to
return:

```rust,noplayground
enum IntervalError {
    StartEndRangeInvalid,
    NonOverlappingInterval,
}
```

## Updating our Interval

First, we'll change the return type for the function `new` and the method
`merge` to a `Result` with the `Ok` variant being an `Interval` and the `Err`
variant the appropriate `IntervalError`:

```rust,noplayground
fn new(start: usize, end: usize) -> Result<Self, IntervalError> {
    if start <= end {
        Ok(Self { start, end })
    } else {
        Err(IntervalError::StartEndRangeInvalid)
    }
}
```

```rust,noplayground
fn merge(&self, other: &Self) -> Result<Self, IntervalError> {
    if self.overlaps(other) {
        Ok(Self {
            start: self.start,
            end: other.end,
        })
    } else {
        Err(IntervalError::NonOverlappingInterval)
    }
}
```

> Observe how an `Interval` is created in `new` as opposed to `merge`. Since the
> parameter names in new precisely match the fields in the `Interval` `struct`
> definition, you can omit the field specifiers. This technique is referred to
> as _field init shorthand_ syntax. [^2]

## Updating Rustle

With the `Interval` changes implemented, we need to update the
`create_intervals` and `merge_intervals` functions to accommodate the `Result`
return type.

```rust,noplayground
fn create_intervals(
    lines: Vec<usize>,
    before_context: usize,
    after_context: usize,
) -> Result<Vec<Interval>, IntervalError> {
    lines
        .iter()
        .map(|line| {
            let start = line.saturating_sub(before_context);
            let end = line.saturating_add(after_context);
            Interval::new(start, end)
        })
        .collect()
}
```

In `create_intervals`, the only change was the return type, which changed from
`Vec<Interval>` to `Result<Vec<Interval>, IntervalError>`. You might wonder why
this works. The concise answer is that the `Result` type implements the
[`FromIterator`] trait.

Imagine an intermediary collection of `Result` values created from calls to
`Interval::new(start, end)`. The `FromIterator` trait implementation allows an
iterator over `Result` values to be collected _into_ a `Result` containing a
collection of the underlying values or an `Err`. [^3] In other words, the value
contained in the `Result` returned by `Interval::new(start, end)` is stored in
the `Vec<Interval>` collection which is then wrapped in a `Result`.

> We'll be exploring traits in the next section.

```rust,noplayground
fn merge_intervals(intervals: Vec<Interval>) -> Vec<Interval> {
    // merge overlapping intervals
    intervals
        .into_iter()
        .coalesce(|p, c| p.merge(&c).or(Err((p, c))))
        .collect()
}
```

In `merge_intervals`, the only change required is to the closure used in
`coalesce`. We attempt to merge two intervals and invoke the [`or`] method of
`Result`. If the merge is successful, returning `Ok`, the value is passed back
to `coalesce`. Otherwise, the `Err((p, c))` value provided to `or` is returned.

> `Result` methods like `map_err`, `or`, and `or_else` are often used in error
> handling because they allow benign errors to be managed while letting
> successful results pass through. Since the `merge` method only merges
> overlapping intervals, we replace the `Err` variant it returns with the tuple
> `(p, c)` needed by `coalesce`.

> Eager vs Lazy Evaluation
>
> Depending on the use case, different `Result` methods may be more efficient.
> It's important to read the documentation to determine the best choice. For
> example, arguments passed to [`or`] are eagerly evaluated. If you're passing
> the result of a function call, it's better to use [`or_else`], which is lazily
> evaluated.

The final change required is in `main`. Since `create_intervals` now returns a
`Result`, we use a `match` expression to check if the operation was successful.
In the case of an `Err`, since it's unrecoverable, we print an error message and
exit.

```rust,noplayground
// create intervals of the form [a,b] with the before/after context
let intervals =
    match create_intervals(match_lines, before_context, after_context) {
        Ok(intervals) => intervals,
        Err(_) => {
            eprintln!("An error occurred while creating intervals");
            exit(1);
        }
    };
```

# Updating Rustle

With our changes in place, our Interval now supports error handling via `Result`
and our rustle program properly handles any errors.

```rust
# #![allow(unused_imports)]
# extern crate regex; // this is needed for the playground
# use itertools::Itertools;
# use regex::Regex;
# use std::fs::File;
# use std::io::Read;
# use std::io::{BufRead, BufReader};
# use std::process::exit;
#
# fn find_matching_lines(lines: &[String], regex: Regex) -> Vec<usize> {
#     lines
#         .iter()
#         .enumerate()
#         .filter_map(|(i, line)| match regex.is_match(line) {
#             true => Some(i),
#             false => None,
#         })
#         .collect() // turns anything iterable into a collection
# }
#
fn create_intervals(
    lines: Vec<usize>,
    before_context: usize,
    after_context: usize,
) -> Result<Vec<Interval>, IntervalError> {
    lines
        .iter()
        .map(|line| {
            let start = line.saturating_sub(before_context);
            let end = line.saturating_add(after_context);
            Interval::new(start, end)
        })
        .collect()
}

fn merge_intervals(intervals: Vec<Interval>) -> Vec<Interval> {
    // merge overlapping intervals
    intervals
        .into_iter()
        .coalesce(|p, c| p.merge(&c).or(Err((p, c))))
        .collect()
}

# fn print_results(intervals: Vec<Interval>, lines: Vec<String>) {
#     for interval in intervals {
#         for (line_no, line) in lines
#             .iter()
#             .enumerate()
#             .take(interval.end + 1)
#             .skip(interval.start)
#         {
#             println!("{}: {}", line_no + 1, line)
#         }
#     }
# }
#
# fn read_file(file: impl Read) -> Vec<String> {
#     BufReader::new(file).lines().map_while(Result::ok).collect()
# }
#
fn main() {
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
#     let mock_file = std::io::Cursor::new(poem);
#
#     // command line arguments
#     let pattern = "(all)|(little)";
#     let before_context = 1;
#     let after_context = 1;
#
#     // attempt to open the file
#     let lines = read_file(mock_file);
#     //let lines = match File::open(filename) {
#     //    // convert the poem into lines
#     //    Ok(file) => read_file(file),
#     //    Err(e) => {
#     //        eprintln!("Error opening {filename}: {e}");
#     //        exit(1);
#     //    }
#     //};
#
#     // compile the regular expression
#     let regex = match Regex::new(pattern) {
#         Ok(re) => re, // bind re to regex
#         Err(e) => {
#             eprintln!("{e}"); // write to standard error
#             exit(1);
#         }
#     };
#
#     // store the 0-based line number for any matched line
#     let match_lines = find_matching_lines(&lines, regex);
#
    // create intervals of the form [a,b] with the before/after context
    let intervals =
        match create_intervals(match_lines, before_context, after_context) {
            Ok(intervals) => intervals,
            Err(_) => {
                eprintln!("An error occurred while creating intervals");
                exit(1);
            }
        };
#
#     // merge overlapping intervals
#     let intervals = merge_intervals(intervals);
#
#     // print the lines
#     print_results(intervals, lines);
}

enum IntervalError {
    StartEndRangeInvalid,
    NonOverlappingInterval,
}

struct Interval {
    start: usize,
    end: usize,
}

impl Interval {
    fn new(start: usize, end: usize) -> Result<Self, IntervalError> {
        if start <= end {
            Ok(Self { start, end })
        } else {
            Err(IntervalError::StartEndRangeInvalid)
        }
    }

    fn overlaps(&self, other: &Interval) -> bool {
        self.end >= other.start
    }

    fn merge(&self, other: &Self) -> Result<Self, IntervalError> {
        if self.overlaps(other) {
            Ok(Self {
                start: self.start,
                end: other.end,
            })
        } else {
            Err(IntervalError::NonOverlappingInterval)
        }
    }
}
```

# Summary

The decision to use the [`Result`] type for error handling in Rust provides a
robust and flexible way of managing errors such as:

1. **Explicit Error Handling**: Using `Result` makes error handling explicit.
   Functions that can fail return a `Result`, which forces the caller to handle
   the potential error, making the code more reliable and less prone to
   unexpected failures.
1. **Recoverable Errors**: The `Result` type allows for recoverable errors. By
   returning a `Result`, the function gives the caller the option to handle the
   error in a way that makes sense for their specific context, rather than
   immediately terminating the program.
1. **Type Safety**: Rust's type system ensures that errors are handled
   correctly. The `Result` type is part of this system, helping to prevent
   common errors like null pointer dereferencing and making the code safer and
   more predictable.
1. **Composability**: The `Result` type implements traits like `FromIterator`,
   which allows for powerful and flexible error handling patterns. This makes it
   easier to work with collections of results and to propagate errors through
   multiple layers of function calls.

Overall, the use of `Result` aligns with Rust's goals of safety, concurrency,
and performance, providing a clear and structured way to handle errors.

# Exercise

- Replace `Result::or` with `Result::map_err` in `merge_intervals`.

  <details>
  <summary>Solution</summary>

  ```rust,noplayground
  fn merge_intervals(intervals: Vec<Interval>) -> Vec<Interval> {
      // merge overlapping intervals
      intervals
          .into_iter()
          .coalesce(|p, c| p.merge(&c).map_err(|_| (p, c)))
          .collect()
  }
  ```

</details>

```rust,editable
#![allow(unused_imports)]
extern crate regex; // this is needed for the playground
use itertools::Itertools;
use regex::Regex;
use std::fs::File;
use std::io::Read;
use std::io::{BufRead, BufReader};
use std::process::exit;

fn find_matching_lines(lines: &[String], regex: Regex) -> Vec<usize> {
    lines
        .iter()
        .enumerate()
        .filter_map(|(i, line)| match regex.is_match(line) {
            true => Some(i),
            false => None,
        })
        .collect() // turns anything iterable into a collection
}

fn create_intervals(
    lines: Vec<usize>,
    before_context: usize,
    after_context: usize,
) -> Result<Vec<Interval>, IntervalError> {
    lines
        .iter()
        .map(|line| {
            let start = line.saturating_sub(before_context);
            let end = line.saturating_add(after_context);
            Interval::new(start, end)
        })
        .collect()
}

fn merge_intervals(intervals: Vec<Interval>) -> Vec<Interval> {
    // merge overlapping intervals
    intervals
        .into_iter()
        .coalesce(|p, c| p.merge(&c).or(Err((p, c))))
        .collect()
}

fn print_results(intervals: Vec<Interval>, lines: Vec<String>) {
    for interval in intervals {
        for (line_no, line) in lines
            .iter()
            .enumerate()
            .take(interval.end + 1)
            .skip(interval.start)
        {
            println!("{}: {}", line_no + 1, line)
        }
    }
}

fn read_file(file: impl Read) -> Vec<String> {
    BufReader::new(file).lines().map_while(Result::ok).collect()
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

    let mock_file = std::io::Cursor::new(poem);

    // command line arguments
    let pattern = "(all)|(little)";
    let before_context = 1;
    let after_context = 1;

    // attempt to open the file
    let lines = read_file(mock_file);
    //let lines = match File::open(filename) {
    //    // convert the poem into lines
    //    Ok(file) => read_file(file),
    //    Err(e) => {
    //        eprintln!("Error opening {filename}: {e}");
    //        exit(1);
    //    }
    //};

    // compile the regular expression
    let regex = match Regex::new(pattern) {
        Ok(re) => re, // bind re to regex
        Err(e) => {
            eprintln!("{e}"); // write to standard error
            exit(1);
        }
    };

    // store the 0-based line number for any matched line
    let match_lines = find_matching_lines(&lines, regex);

    // create intervals of the form [a,b] with the before/after context
    let intervals =
        match create_intervals(match_lines, before_context, after_context) {
            Ok(intervals) => intervals,
            Err(_) => {
                eprintln!("An error occurred while creating intervals");
                exit(1);
            }
        };

    // merge overlapping intervals
    let intervals = merge_intervals(intervals);

    // print the lines
    print_results(intervals, lines);
}

enum IntervalError {
    StartEndRangeInvalid,
    NonOverlappingInterval,
}

struct Interval {
    start: usize,
    end: usize,
}

impl Interval {
    fn new(start: usize, end: usize) -> Result<Self, IntervalError> {
        if start <= end {
            Ok(Self { start, end })
        } else {
            Err(IntervalError::StartEndRangeInvalid)
        }
    }

    fn overlaps(&self, other: &Interval) -> bool {
        self.end >= other.start
    }

    fn merge(&self, other: &Self) -> Result<Self, IntervalError> {
        if self.overlaps(other) {
            Ok(Self {
                start: self.start,
                end: other.end,
            })
        } else {
            Err(IntervalError::NonOverlappingInterval)
        }
    }
}
```

# Next

With our Interval complete, let's make it a module!

[^1]: Refer to the documentation on [enum values].

[^2]: Refer to the documentation on _[field init shorthand]_ syntax.

[^3]: Refer to the documentation on [Collecting into a `Result`] for detailed
    explanation.

[collecting into a `result`]: https://doc.rust-lang.org/std/result/#collecting-into-result
[enum]: https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html
[enum values]: https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html#enum-values
[enumeration]: https://doc.rust-lang.org/reference/items/enumerations.html
[field init shorthand]: https://doc.rust-lang.org/book/ch05-01-defining-structs.html#using-the-field-init-shorthand
[`fromiterator`]: https://doc.rust-lang.org/std/iter/trait.FromIterator.html
[`or_else`]: https://doc.rust-lang.org/std/result/enum.Result.html#method.or_else
[`or`]: https://doc.rust-lang.org/std/result/enum.Result.html#method.or
[`result`]: https://doc.rust-lang.org/std/result/enum.Result.html
