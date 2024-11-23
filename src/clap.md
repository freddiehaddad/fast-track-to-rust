# Command Line Arguments

In the [`Cargo.toml`] section, you were asked to add the [`clap`] crate as a
dependency to support command line argument parsing for our grep program. With
our current understanding of attributes, it's an ideal time to utilize this
crate by extending our grep program to handle command line arguments.

One method for defining command line arguments with `clap` involves using custom
attributes. This is why we specified the derive feature in the dependency.

If you didn't complete the exercise and are working through this course locally,
you can add the dependency to the grep crate with:

```console
$ cargo add clap --features derive
```

## Advanced Attributes

The topic of [macro attributes] and [derive macro helper attributes] is quite
advanced and beyond the scope of this course. Therefore, we won't delve into
their inner workings in depth. However, keep this section in mind as you
continue your Rust journey, because the time will likely come when you will need
to implement similar attributes yourself for code generation.

## Extending Grep

We're going to add command line argument support to our program. However, we'll
continue to mock the command line arguments to keep developing this course
online. After configuring `clap`, this will be the auto-generated help (i.e.,
typing `grep --help` in the terminal).

```console
$ grep.exe --help
Usage: grep.exe [OPTIONS] <PATTERN> [FILES]...

Arguments:
  <PATTERN>   The regular expression to match
  [FILES]...  List of files to search

Options:
  -l, --line-number           Prefix each line of output with the 1-based line
                              number within its input file
  -b, --before-context <num>  Print num lines of trailing context before matching
                              lines [default: 0]
  -a, --after-context <num>   Print num lines of trailing context after matching
                              lines [default: 0]
  -h, --help                  Print help
  -V, --version               Print version
```

> The `CLAP` documentation is very extensive and includes plenty of examples to
> learn from. If you find yourself building a CLI application, you will most
> certainly want to reference the [documentation].

Here's the updated version of grep with command line argument support added:

````rust
# #![allow(unused)]
# extern crate clap; // this is needed for the playground
# extern crate itertools; // this is needed for the playground
# extern crate regex; // this is needed for the playground
use clap::{Parser, Subcommand};
# use interval::{Interval, IntervalError};
# use itertools::Itertools;
# use regex::Regex;
# use std::fs::File;
# use std::io::{BufRead, BufReader, Read};
use std::path::PathBuf;
# use std::process::exit;
#
# fn create_intervals(
#     before_context: usize,
#     after_context: usize,
#     match_lines: Vec<usize>,
#     lines: &[String],
# ) -> Result<Vec<Interval<usize>>, IntervalError> {
#     match_lines
#         .iter()
#         .map(|line| {
#             let start = line.saturating_sub(before_context);
#             let end = line.saturating_add(after_context);
#             Interval::new(start, end)
#         })
#         .collect()
# }
#
# fn merge_intervals(intervals: Vec<Interval<usize>>) -> Vec<Interval<usize>> {
#     intervals
#         .into_iter()
#         .coalesce(|p, c| p.merge(&c).or(Err((p, c))))
#         .collect()
# }

fn print_matches(
    intervals: Vec<Interval<usize>>,
    line_numbers: bool,
    lines: &[String],
) {
    for interval in intervals {
        for (line_no, line) in lines
            .iter()
            .enumerate()
            .take(interval.end + 1)
            .skip(interval.start)
        {
            if line_numbers {
                print!("{}: ", line_no + 1);
            }
            println!("{}", line);
        }
    }
}
#
# fn read_file(file: impl Read) -> Vec<String> {
#     BufReader::new(file).lines().map_while(Result::ok).collect()
# }

#[derive(Parser)]
#[command(version, about, long_about = None)]
struct Cli {
    /// Prefix each line of output with the 1-based line number within its
    /// input file.
    #[arg(short, long, default_value_t = false)]
    line_number: bool,

    /// Print num lines of trailing context before matching lines.
    #[arg(short, long, default_value_t = 0, value_name = "num")]
    before_context: u8,

    /// Print num lines of trailing context after matching lines.
    #[arg(short, long, default_value_t = 0, value_name = "num")]
    after_context: u8,

    /// The regular expression to match.
    #[arg(required = true)]
    pattern: String,

    /// List of files to search.
    #[arg(required = true)]
    files: Vec<PathBuf>,
}

fn main() {
    // let cli = Cli::parse(); // Use this version for production.
    // mock command line arguments
    let cli = match Cli::try_parse_from([
        "grep", // executable name
        "--line-number",
        "--before-context",
        "1",
        "--after-context",
        "1",
        "little",   // pattern
        "poem.txt", // file
    ]) {
        Ok(cli) => cli,
        Err(e) => {
            eprintln!("Error parsing command line arguments: {e:?}");
            exit(1);
        }
    };
#
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

    // get values from command line arguments
    let line_numbers = cli.line_number;
    let pattern = &cli.pattern;
    let before_context = cli.before_context as usize;
    let after_context = cli.after_context as usize;
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
#     let intervals = match create_intervals(
#         before_context,
#         after_context,
#         match_lines,
#         &lines,
#     ) {
#         Ok(intervals) => intervals,
#         Err(_) => {
#             eprintln!("An error occurred while creating intervals");
#             exit(1);
#         }
#     };
#
#     let intervals = merge_intervals(intervals);

    print_matches(intervals, line_numbers, &lines);
}
#
# pub mod interval {
#     /// A list specifying general categories of Interval errors.
#     #[derive(Debug)]
#     pub enum IntervalError {
#         /// Start is not less than or equal to end
#         StartEndRangeInvalid,
#         /// Two intervals to be merged do not overlap
#         NonOverlappingInterval,
#     }
#
#     /// A closed-interval [`start`, `end`] type used for representing a range of
#     /// values between `start` and `end` inclusively.
#     ///
#     /// # Examples
#     ///
#     /// You can create an `Interval` using `new`.
#     ///
#     /// ```rust
#     /// let interval = Interval::new(1, 10).unwrap();
#     /// assert_eq!(interval.start, 1);
#     /// assert_eq!(interval.end, 10);
#     /// ```
#     #[derive(Debug, PartialEq)]
#     pub struct Interval<T> {
#         pub start: T,
#         pub end: T,
#     }
#
#     use std::fmt;
#     impl<T: fmt::Display> fmt::Display for Interval<T> {
#         fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> fmt::Result {
#             write!(f, "[{}, {}]", self.start, self.end)
#         }
#     }
#
#     use std::cmp::Ordering;
#     impl<T: PartialEq + PartialOrd> PartialOrd for Interval<T> {
#         fn partial_cmp(&self, other: &Self) -> Option<Ordering> {
#             if self == other {
#                 Some(Ordering::Equal)
#             } else if self.end < other.start {
#                 Some(Ordering::Less)
#             } else if self.start > other.end {
#                 Some(Ordering::Greater)
#             } else {
#                 None // Intervals overlap
#             }
#         }
#     }
#
#     impl<T: Copy + PartialOrd> Interval<T> {
#         /// Creates a new `Interval` set to `start` and `end`.
#         ///
#         /// # Examples
#         ///
#         /// ```rust
#         /// let interval = Interval::new(1, 10).unwrap();
#         /// assert_eq!(interval.start, 1);
#         /// assert_eq!(interval.end, 10);
#         /// ```
#         pub fn new(start: T, end: T) -> Result<Self, IntervalError> {
#             if start <= end {
#                 Ok(Self { start, end })
#             } else {
#                 Err(IntervalError::StartEndRangeInvalid)
#             }
#         }
#
#         /// Checks if two intervals overlap. Overlapping intervals have at least
#         /// one point in common.
#         ///
#         /// # Examples
#         ///
#         /// ```rust
#         /// let a = Interval::new(1, 3).unwrap();
#         /// let b = Interval::new(3, 5).unwrap();
#         /// assert_eq!(a.overlaps(&b), true);
#         /// assert_eq!(b.overlaps(&a), true);
#         /// ```
#         ///
#         /// ```rust
#         /// let a = Interval::new(1, 5).unwrap();
#         /// let b = Interval::new(2, 4).unwrap();
#         /// assert_eq!(a.overlaps(&b), true);
#         /// assert_eq!(b.overlaps(&a), true);
#         /// ```
#         ///
#         /// ```rust
#         /// let a = Interval::new(1, 3).unwrap();
#         /// let b = Interval::new(4, 6).unwrap();
#         /// assert_eq!(a.overlaps(&b), false);
#         /// assert_eq!(b.overlaps(&a), true);
#         /// ```
#         pub fn overlaps(&self, other: &Interval<T>) -> bool {
#             self.end >= other.start
#         }
#
#         /// Merges two intervals returning a new `Interval`.
#         ///
#         /// The merged `Interval` range includes the union of ranges from each
#         /// `Interval`.
#         ///
#         /// # Examples
#         ///
#         /// ```rust
#         /// let a = Interval::new(1, 3).unwrap();
#         /// let b = Interval::new(3, 5).unwrap();
#         /// let c = a.merge(&b).unwrap();
#         /// assert_eq!(c.start, 1);
#         /// assert_eq!(c.end, 5);
#         /// ```
#         pub fn merge(&self, other: &Self) -> Result<Self, IntervalError> {
#             if self.overlaps(other) {
#                 Ok(Self {
#                     start: self.start,
#                     end: other.end,
#                 })
#             } else {
#                 Err(IntervalError::NonOverlappingInterval)
#             }
#         }
#     }
# }
````

# Summary

We made some minor code changes along with configuring `clap`. Most of it should
be straightforward by now, so we'll focus on the new aspects:

- The [`PathBuf`] module facilitates cross-platform path manipulation. Since our
  grep program requires files to search, it's better to use `PathBuf` for
  handling that input.
- We updated `print_matches` to take a new boolean argument, `line_numbers`,
  which specifies whether or not to print line numbers in the output.
- We used `clap` attributes to derive our command line arguments. The `clap`
  [documentation] covers these attributes in full detail.
- In `main`, we mock command line arguments with an array of string slices that
  gets parsed by `clap`.
  > The `match` expression is for development purposes. If we made any mistakes
  > in our mock command line arguments, we would catch the error and print it.
- Since we defined the before- and after-context variables as `u8` in the `Cli`
  structure but use `usize` throughout the code, an explicit cast using [`as`]
  is necessary when assigning these values to local variables.

# Next

Onward to concurrency!

[`Cargo.toml`]: cargo_toml.md
[`clap`]: https://crates.io/crates/clap
[macro attributes]:
  https://doc.rust-lang.org/reference/procedural-macros.html#attribute-macros
[derive macro helper attributes]:
  https://doc.rust-lang.org/reference/procedural-macros.html#derive-macro-helper-attributes
[documentation]: https://crates.io/crates/clap
[`Pathbuf`]: https://doc.rust-lang.org/std/path/struct.PathBuf.html
[`as`]: https://doc.rust-lang.org/std/keyword.as.html
