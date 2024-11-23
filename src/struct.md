# Struct

We're going to create a type to represent an interval. Rust follows a
standardized naming convention for types, where structures use
`UpperCamelCase`[^1] for "type-level" constructs. Therefore, we'll name our
`struct` `Interval`.

## Defining our Struct

Our closed interval is going to represent the start and end of the lines to
print:

```rust,noplayground
struct Interval {
    start: usize,
    end: usize,
}
```

## Defining Methods

Methods are added by defining them inside an `impl` block:

```rust,noplayground
impl Interval {
    // Methods definitions
}
```

## The `new` Function

As it is common convention in Rust to define a function called `new` for
creating an object, we'll begin by defining one to return an `Interval`.

```rust,noplayground
impl Interval {
    fn new(start: usize, end: usize) -> Self {
        Self { start, end }
    }
}
```

> The keyword `Self`
>
> The `Self` keywords in the return type and the body of the function are
> aliases for the type specified after the `impl` keyword, which in this case is
> `Interval`. While the type can be explicitly stated, the common convention is
> to use `Self`.

> Exclusion of the <code>return</code> keyword and <code>;</code>
>
> `return` is not needed when the returned value is the last expression in a
> function. In this case the `;` is omitted.[^2]

## Methods

Recall from our grep program that we merged overlapping intervals to prevent
printing the same line multiple times. It would be useful to create a method to
check if two intervals overlap and another method to merge overlapping
intervals. Let's outline these methods!

```rust,noplayground
fn overlaps(&self, other: &Interval) -> bool {
    todo!();
}

fn merge(&self, other: &Self) -> Self {
    todo!();
}
```

> The `todo!()` and `unimplemented!()` macros can be useful if you are
> prototyping and just want a placeholder to let your code pass type analysis.

### The `self` Method Receiver

You're probably accustomed to using the implicit `this` pointer (a hidden first
parameter) in your class methods. In Rust, the [`self`] keyword is used to
represent the receiver of a method and the convention is to omit the type for
this parameter. Depending on the intended behavior, it can be specified as
`self`, `&self` or `&mut self`.[^3]

> Omitting the type after `self` is syntactic sugar. It's short for
> `self: Self`. As `Self` is an alias to the actual type, the full expansion
> would be `self: Interval`, `self: &Interval` or `self: &mut Interval`.

## Implementing the Methods

Remember that our grep program processes lines sequentially. This allows us to
optimize the detection of overlapping intervals. However, this approach limits
the versatility of our `Interval` type. As an exercise, you can work on making
it more generic.

### `overlaps`

The `overlaps` method is fairly straightforward. We check if the `end` of the
first interval is greater than or equal to the `start` of the next interval. The
only caveat is the order of the comparison.

```rust,noplayground
fn overlaps(&self, other: &Interval) -> bool {
    self.end >= other.start
}
```

### `merge`

The `merge` method returns a new `Interval` using the `start` of the first
interval and the `end` of the second. The same caveat applies: the receiver must
be the interval that comes first in the sequence.

In both cases, an immutable borrow for both intervals is sufficient since we do
not need to mutate either value.

```rust,noplayground
fn merge(&self, other: &Self) -> Self {
    Interval::new(self.start, other.end)
}
```

### Implementing `merge_intervals`

Rust programmers coming from a C++ background might be inclined to implement
this function using some variation of the following algorithm:

```rust,noplayground
fn merge_intervals(intervals: Vec<Interval>) -> Vec<Interval> {
    let mut merged_intervals: Vec<Interval> = Vec::new();

    let mut iter = intervals.into_iter();
    match iter.next() {
        Some(interval) => merged_intervals.push(interval),
        None => return merged_intervals,
    };

    for interval in iter {
        if let Some(previous_interval) = merged_intervals.last() {
            if previous_interval.overlaps(&interval) {
                let new_interval = previous_interval.merge(&interval);
                merged_intervals.pop();
                merged_intervals.push(new_interval);
            } else {
                merged_intervals.push(interval);
            }
        }
    }

    merged_intervals
}
```

While functionally correct, Rust features powerful crates that can make
implementing this behavior more concise. One such crate is [`Itertools`], which
provides extra iterator adaptors. To use this crate, specify it as a dependency
in `Crates.toml` and include it in `main.rs` with `use itertools::Itertools`.
Let's see how the [`coalesce`] adaptor can simplify the code:

```rust,noplayground
use itertools::Itertools;

fn merge_intervals(intervals: Vec<Interval>) -> Vec<Interval> {
    intervals
        .into_iter()
        .coalesce(|p, c| {
            if p.overlaps(&c) {
                Ok(p.merge(&c))
            } else {
                Err((p, c))
            }
        })
        .collect()
}
```

> `into_iter`
>
> In both implementations, we use `into_iter`, which creates a consuming
> iterator that moves each value out of the vector from start to end. This is
> another example of how we can take advantage of move semantics.
>
> NOTE: Because the iterator consumes the data, the vector cannot be used
> afterward.

## Updating Grep

It's time to update our Grep program to utilize our new type. Additionally, a
few minor changes have been made to enhance the design, demonstrate some
additional language features, and leverage move semantics. Give the program a
run!

```rust
# extern crate itertools; // this is needed for the playground
# extern crate regex; // this is needed for the playground
use itertools::Itertools;
# use regex::Regex;
# use std::fs::File;
# use std::io::{BufRead, BufReader, Read};
# use std::process::exit;

fn create_intervals(
    before_context: usize,
    after_context: usize,
    match_lines: Vec<usize>,
    lines: &[String],
) -> Vec<Interval> {
    match_lines
        .iter()
        .map(|line| {
            let start = line.saturating_sub(before_context);
            let end = line.saturating_add(after_context);
            Interval::new(start, end)
        })
        .collect()
}

fn merge_intervals(intervals: Vec<Interval>) -> Vec<Interval> {
    intervals
        .into_iter()
        .coalesce(|p, c| {
            if p.overlaps(&c) {
                Ok(p.merge(&c))
            } else {
                Err((p, c))
            }
        })
        .collect()
}

fn print_matches(intervals: Vec<Interval>, lines: &[String]) {
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

# fn read_file(file: impl Read) -> Vec<String> {
#     BufReader::new(file).lines().map_while(Result::ok).collect()
# }
#
fn main() {
#     // let filename = "poem.txt";
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
#     let mock_file = std::io::Cursor::new(poem);
#     let lines = read_file(mock_file);
#
#     let pattern = "little";
#     let before_context = 1;
#     let after_context = 1;
#
#     // // attempt to open the file
#     // let lines = match File::open(filename) {
#     //     Ok(file) => read_file(file),
#     //     Err(e) => {
#     //         eprintln!("Error opening {filename}: {e}");
#     //         exit(1);
#     //     }
#     // };
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
#     let match_lines: Vec<_> = lines
#         .iter()
#         .enumerate()
#         .filter_map(|(i, line)| match regex.is_match(line) {
#             true => Some(i),
#             false => None,
#         })
#         .collect(); // turns anything iterable into a collection
#
#     // exit early if no matches were found
#     if match_lines.is_empty() {
#         return;
#     }
#
#     // create intervals of the form [a,b] with the before/after context
#     let intervals =
#         create_intervals(before_context, after_context, match_lines, &lines);
#
    let intervals = merge_intervals(intervals);

    print_matches(intervals, &lines);
}

struct Interval {
    start: usize,
    end: usize,
}

impl Interval {
    fn new(start: usize, end: usize) -> Self {
        Self { start, end }
    }

    fn overlaps(&self, other: &Interval) -> bool {
        self.end >= other.start
    }

    fn merge(&self, other: &Self) -> Self {
        Interval::new(self.start, other.end)
    }
}
```

## Summarizing the Changes

Our grep program now makes use of our custom type and includes a few other
enhancements.

Let's review the changes:

Merging intervals has been refactored into a new function called
`merge_intervals`.

Printing the results was also refactored into a new function called
`print_matches` with minor changes. Since we are making use of the `Interval`
type, we update the `for` loop to use the `start` and `end` values of the
`Interval`.

> Each interval is _dropped_ at the end of the loop iteration when written as
> `for interval in intervals`. If the loop were written as
> `for interval in &intervals`, we would _borrow_ each value. The same applies
> if we had written the loop as `for interval in intervals.iter()`. The former
> is syntactic sugar for the latter.[^4]

[RFC 430]:
  https://github.com/rust-lang/rfcs/blob/master/text/0430-finalizing-naming-conventions.md
[`Itertools`]: https://docs.rs/itertools/latest/itertools
[`coalesce`]:
  https://docs.rs/itertools/latest/itertools/trait.Itertools.html#method.coalesce
[`self`]: https://doc.rust-lang.org/std/keyword.self.html

---

[^1]: Casing conforms to [RFC 430] (C-CASE).

[^2]:
    Exclusion of `return` is discussed
    [here](https://doc.rust-lang.org/std/keyword.return.html).

[^3]:
    Method syntax is covered in full detail
    [here](https://doc.rust-lang.org/book/ch05-03-method-syntax.html).

[^4]:
    Details about consuming collections with `for in` and `into_iter` can be
    found
    [here](https://doc.rust-lang.org/rust-by-example/flow_control/for.html).
