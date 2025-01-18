# File I/O

Our rustle program wouldn't be complete without the ability to search text
files. Given the potential for I/O errors, adding this capability now is
convenient as we explore error handling and the `Result` type. This also
introduces us to additional packages in Rust's standard library.

> Up to this point, we've been able to use string literals in our rustle program
> because dynamic memory allocation wasn't needed. However, now that we will be
> reading from a file, dynamic memory allocation becomes necessary. The string
> slice is no longer sufficient, so we need to utilize the [`String`] [^1] type
> in Rust.

> Storing Data on the Heap
>
> Should you find yourself needing to allocate memory directly on the heap, the
> [`Box`] type is commonly used. You can find numerous examples on its usage in
> the [documentation on the `Box` type].

Let's start by creating a function that reads a file and returns a vector of
strings (`Vec<String>`) where each string represents a line. Here is the
function signature:[^2] [^3]

```rust,noplayground
fn read_file(file: File) -> Vec<String> {
    todo!(); // see the footnote [^3]
}
```

This is the code that we'll add to the `read_file` function:

```rust,noplayground
BufReader::new(file).lines().map_while(Result::ok).collect()
```

The `read_file` function accepts a file handle and utilizes [`BufReader`] to
efficiently read the file line by line, storing each line in a vector of strings
(`Vec<String>`), which it then returns to the caller.

> Many less efficient methods for reading a file and storing the results in a
> collection typically involve iterating over each line, converting it to a
> string, and then pushing the string into a vector. This approach requires
> intermediate memory allocations, which can become costly for large files.
> Additionally, each line read from the file potentially involves a system call.
> The [`BufReader`] uses an internal buffer to read large chunks of data from
> the file, minimizing both memory allocations and system calls.

The modifications to the `main` function:

```rust,noplayground
fn main() {
    // command line arguments
#     let pattern = "all";
#     let before_context = 1;
#     let after_context = 1;
    let filename = "poem.txt";

    // attempt to open the file
    let lines = match File::open(filename) {
        // convert the poem into lines
        Ok(file) => read_file(file),
        Err(e) => {
            eprintln!("Error opening {filename}: {e}");
            exit(1);
        }
    };
#
#     // store the 0-based line number for any matched line
#     let match_lines = find_matching_lines(&lines, pattern);
#
#     // create intervals of the form [a,b] with the before/after context
#     let mut intervals =
#         create_intervals(match_lines, before_context, after_context);
#
#     // merge overlapping intervals
#     merge_intervals(&mut intervals);
#
#     // print the lines
#     print_results(intervals, lines);
}
```

## Unpacking the Code

There's a lot going on here, so let's break it down step by step.

### `read_file`

```rust,noplayground
fn read_file(file: File) -> Vec<String> {
    BufReader::new(file).lines().map_while(Result::ok).collect()
}
```

1. **`BufReader`**: `BufReader::new(file)` creates a buffered reader from the
   provided `File`. This helps in efficiently reading the file line by line.
1. **`lines()`**: The `lines()` method on `BufReader` returns an iterator over
   the lines in the file. Because reading from a file can file, each line is
   wrapped in a `Result`, which can be either `Ok` (containing the line) or
   `Err` (containing an error).
1. **`map_while(Result::ok)`**: The `map_while` method is used to transform the
   iterator. It applies the `Result::ok` function to each item, which converts
   `Ok(line)` to `Some(line)` and `Err(_)` to `None`. The iteration stops when
   the first `None` is encountered. Here are the relevant parts from the
   [source code](https://doc.rust-lang.org/std/result/enum.Result.html), cleaned
   up for readability:

   ```rust,noplayground
   pub enum Result<T, E> {
      Ok(T),
      Err(E),
   }

   impl<T, E> Result<T, E> {
       pub fn ok(self) -> Option<T> {
           match self {
               Ok(x) => Some(x),
               Err(_) => None,
           }
       }
   }
   ```

   This conversion is necessary because the map method requires the closure to
   return an `Option`. Converting `Err` to `None` drops the error value and
   causes `map_while` to stop yielding.

1. **`collect()`**: The `collect()` method gathers all the `Some(line)` values
   into a `Vec<String>` that gets returned to the caller.

### `main`

In the `main` function, we attempt to open a file, which can fail for various
reasons. If the `Result` is `Ok`, we call `read_file` with the file value. Since
we don't need the file handle afterward, borrowing isn't necessary. If an error
occurs while opening the file, we use the `eprintln!` macro to print the error
to standard error and then exit.

## Putting it All Together

Here are the changes with the unrelated parts of the program hidden:

```rust,noplayground
use std::fs::File;
use std::io::{BufRead, BufReader};
use std::process::exit;
#
# fn find_matching_lines(lines: &[String], pattern: &str) -> Vec<usize> {
#     lines
#         .iter()
#         .enumerate()
#         .filter_map(|(i, line)| match line.contains(pattern) {
#             true => Some(i),
#             false => None,
#         })
#         .collect() // turns anything iterable into a collection
# }
#
# fn create_intervals(
#     lines: Vec<usize>,
#     before_context: usize,
#     after_context: usize,
# ) -> Vec<(usize, usize)> {
#     lines
#         .iter()
#         .map(|line| {
#             (
#                 line.saturating_sub(before_context),
#                 line.saturating_add(after_context),
#             )
#         })
#         .collect()
# }
#
# fn merge_intervals(intervals: &mut Vec<(usize, usize)>) {
#     // merge overlapping intervals
#     intervals.dedup_by(|next, prev| {
#         if prev.1 < next.0 {
#             false
#         } else {
#             prev.1 = next.1;
#             true
#         }
#     })
# }
#
# fn print_results(intervals: Vec<(usize, usize)>, lines: Vec<String>) {
#     for (start, end) in intervals {
#         for (line_no, line) in
#             lines.iter().enumerate().take(end + 1).skip(start)
#         {
#             println!("{}: {}", line_no + 1, line)
#         }
#     }
# }

fn read_file(file: File) -> Vec<String> {
    BufReader::new(file).lines().map_while(Result::ok).collect()
}

fn main() {
    // command line arguments
#     let pattern = "all";
#     let before_context = 1;
#     let after_context = 1;
    let filename = "poem.txt";

    // attempt to open the file
    let lines = match File::open(filename) {
        // convert the poem into lines
        Ok(file) => read_file(file),
        Err(e) => {
            eprintln!("Error opening {filename}: {e}");
            exit(1);
        }
    };
#
#     // store the 0-based line number for any matched line
#     let match_lines = find_matching_lines(&lines, pattern);
#
#     // create intervals of the form [a,b] with the before/after context
#     let mut intervals =
#         create_intervals(match_lines, before_context, after_context);
#
#     // merge overlapping intervals
#     merge_intervals(&mut intervals);
#
#     // print the lines
#     print_results(intervals, lines);
}
```

> Don't forget, you can reveal the hidden parts by clicking _Show hidden lines_.

# Summary

- Rust requires acknowledging and handling errors before code compilation,
  ensuring robustness.
- Errors are categorized into recoverable (e.g., file not found) and
  unrecoverable (e.g., out-of-bounds access).
- Rust uses `Result<T, E>` for recoverable errors and `panic!` for unrecoverable
  errors, unlike other languages that use exceptions.

# Next

To continue using the Rust Playground, opening an actual file isn't going to
work. Let's see how we can leverage an in-memory buffer to represent an open
file.

[`String`]: https://doc.rust-lang.org/rust-by-example/std/str.html
[`Box`]: https://doc.rust-lang.org/std/boxed/struct.Box.html
[documentation on the `Box` type]:
  https://doc.rust-lang.org/book/ch15-01-box.html
[`todo!()`]: https://doc.rust-lang.org/std/macro.todo.html
[`unimplemented!()`]: https://doc.rust-lang.org/std/macro.unimplemented.html
[`BufReader`]: https://doc.rust-lang.org/std/io/struct.BufReader.html

---

[^1]:
    Strings are implemented as `Vec<u8>` in Rust. Reference the
    [API](https://doc.rust-lang.org/stable/std/string/index.html) for details.

[^2]:
    Unfortunately, the Rust Playground doesn't support opening files, so you'll
    need to run this part of the code on your local machine.

[^3]:
    Rust offers several useful macros that are handy for developing and
    prototyping your program. [`todo!()`] is one of them, and another is
    [`unimplemented!()`].

[^4]:
    Unlike many object-oriented programming languages that use `this`, Rust uses
    `self`.
