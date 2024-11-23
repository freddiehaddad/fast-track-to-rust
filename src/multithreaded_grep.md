# Multithreaded Grep

With our understanding of scoped vs non-scoped threads, we are now prepared to
correctly update Grep to process each file specified on the command line in a
separate thread.

Here's the updated version of the program with the changes visible.

````rust
# #![allow(unused)]
# extern crate clap; // this is needed for the playground
# extern crate itertools; // this is needed for the playground
# extern crate regex; // this is needed for the playground
# use clap::{Parser, Subcommand};
# use interval::{Interval, IntervalError};
# use itertools::Itertools;
# use regex::Regex;
use std::collections::HashMap;
# use std::fs::File;
# use std::io::{BufRead, BufReader, Read};
# use std::path::PathBuf;
# use std::process::exit;
use std::thread;

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
#
# fn print_matches(
#     intervals: Vec<Interval<usize>>,
#     line_numbers: bool,
#     lines: Vec<String>,
# ) {
#     for interval in intervals {
#         for (line_no, line) in lines
#             .iter()
#             .enumerate()
#             .take(interval.end + 1)
#             .skip(interval.start)
#         {
#             if line_numbers {
#                 print!("{}: ", line_no + 1);
#             }
#             println!("{}", line);
#         }
#     }
# }
#
# fn read_file(file: impl Read) -> Vec<String> {
#     BufReader::new(file).lines().map_while(Result::ok).collect()
# }
#
# #[derive(Parser)]
# #[command(version, about, long_about = None)]
# struct Cli {
#     /// Prefix each line of output with the 1-based line number within its
#     /// input file.
#     #[arg(short, long, default_value_t = false)]
#     line_number: bool,
#
#     /// Print num lines of trailing context before matching lines.
#     #[arg(short, long, default_value_t = 0, value_name = "num")]
#     before_context: u8,
#
#     /// Print num lines of trailing context after matching lines.
#     #[arg(short, long, default_value_t = 0, value_name = "num")]
#     after_context: u8,
#
#     /// The regular expression to match.
#     #[arg(required = true)]
#     pattern: String,
#
#     /// List of files to search.
#     #[arg(required = true)]
#     files: Vec<PathBuf>,
# }
#
struct GrepSuccess {
    intervals: Vec<Interval<usize>>,
    lines: Vec<String>,
}

struct GrepFailure {
    error: String,
}

fn main() {
#     // let cli = Cli::parse(); // Use this version for production.
    // mock command line arguments
    let cli = match Cli::try_parse_from([
#        // executable
#        "grep",
#        // args
#        "--line-number",
#        "--before-context",
#        "1",
#        "--after-context",
#        "1",
        // pattern
        "me",
        // files(s)...
        "poem.txt",
        "bad_file.txt", // intended failure
        "scoped-threads.txt",
    ]) {
        Ok(cli) => cli,
        Err(e) => {
            eprintln!("Error parsing command line arguments: {e:?}");
            exit(1);
        }
    };

    // mock disk for opening files
    let mock_disk = HashMap::from([
        (
            "poem.txt",
            r"I have a little shadow that goes in and out with me,
And what can be the use of him is more than I can see.
He is very, very like me from the heels up to the head;
And I see him jump before me, when I jump into my bed.

The funniest thing about him is the way he likes to grow -
Not at all like proper children, which is always very slow;
For he sometimes shoots up taller like an india-rubber ball,
And he sometimes gets so little that there’s none of him at all.",
        ),
        (
            "scoped-threads.txt",
            r"When we work with scoped threads, the compiler can clearly see,
if the variables we want to use will be avilable to me.
Because of this visiblity, I'm runtime error free!
And issues in my code will be exposed by rustc.,
If this sort of safety is provided at native speeds,
there's simply no compelling case to stick with cpp!",
        ),
    ]);

#     // get values from command line arguments
#     let line_number = cli.line_number;
#     let pattern = &cli.pattern;
#     let before_context = cli.before_context as usize;
#     let after_context = cli.after_context as usize;
#     let files = cli.files;
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
    thread::scope(|s| {
        let handles: Vec<_> = files
            .iter()
            .map(|file| {
                let filename = match file.to_str() {
                    Some(filename) => filename,
                    None => {
                        return Err(GrepFailure {
                            error: format!(
                                "Invalid filename: {}",
                                file.display()
                            ),
                        })
                    }
                };

#                 // attempt to open the file
#                 //let lines = match File::open(filename) {
#                 //    Ok(file) => read_file(file),
#                 //    Err(e) => {
#                 //        return Err(GrepFailure {
#                 //            error: format!("Error opening file: {e}"),
#                 //        })
#                 //    }
#                 //};
#
                if !mock_disk.contains_key(filename) {
                    return Err(GrepFailure {
                        error: format!("File not found: {}", filename),
                    });
                }

                Ok(filename)
            })
            .map_ok(|filename| {
                // only spawn a thread for accessible file.
                s.spawn(|| {
                    let contents = mock_disk.get(filename).unwrap();
                    let mock_file = std::io::Cursor::new(contents);
                    let lines = read_file(mock_file);

                    // store the 0-based line number for any matched line
                    let match_lines: Vec<_> = lines
                        .iter()
                        .enumerate()
                        .filter_map(|(i, line)| match regex.is_match(line) {
                            true => Some(i),
                            false => None,
                        })
                        .collect(); // turns anything iterable into a collection

                    // create intervals of the form [a,b] with the before/after
                    // context
                    let intervals = match create_intervals(
                        before_context,
                        after_context,
                        match_lines,
                        &lines,
                    ) {
                        Ok(intervals) => intervals,
                        Err(_) => return Err(GrepFailure {
                            error: String::from(
                                "An error occurred while creating intervals",
                            ),
                        }),
                    };

                    let intervals = merge_intervals(intervals);
                    Ok(GrepSuccess { intervals, lines })
                })
            })
            .collect();

        // process all the results
        for handle in handles {
            let result = match handle {
                Ok(scoped_join_handle) => scoped_join_handle,
                Err(e) => {
                    eprintln!("{}", e.error);
                    continue;
                }
            };

            if let Ok(result) = result.join() {
                match result {
                    Ok(result) => print_matches(
                        result.intervals,
                        line_number,
                        result.lines,
                    ),
                    Err(e) => eprintln!("{}", e.error),
                };
            };
        }
    });
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

> The error message for the bad file doesn't appear in the playground output
> because standard error output isn't captured.

Our grep program is now multithreaded and processes all the input files in
parallel! Let's walk through the code changes.

### `mock_disk`

The `mock_disk` [`HashMap`] is used to simulate disk access to see if a file
exists as is visible. We use the filename as the key and the contents of the
file is the value. This idea helps with testing and development, but it's usage
here is purely for Rust playground compatibility.

### `thread::scope`

1. Iterate over all files specified on the command line using the `map` iterator
   adapter, which returns a `Result`. The `Ok` variant holds the filename, while
   the `Error` variant holds any file access errors.
1. The `map_ok` iterator adapter processes each `Result`, calling the provided
   closure on any `Ok` values, allowing us to ignore files that couldn't be
   opened. The provided `Scope` (`s`) spawns one thread per file for processing.
   The closure returns a `Result`: `Err` with an error message in a
   `GrepFailure` struct if processing fails, or `Ok` with a `GrepSuccess` struct
   containing intervals and lines from the input file if successful.
1. Use `collect` to create a vector (`Vec`) of results from each file iteration,
   binding it to `handles`.
1. Finally, iterate over the elements in the `handles` vector using a for loop.
   Print any errors to standard error, and pass successful pattern matching
   results to the `print_matches` function for output to standard output.

# Summary

- Ownership and type systems are powerful tools for managing memory safety and
  concurrency issues. By leveraging ownership and type checking, many
  concurrency errors in Rust are caught at compile time rather than at runtime.
- Unlike non-scoped threads, scoped threads can borrow non-`'static` data
  because the scope ensures all threads are joined before it ends.

[`HashMap`]: https://doc.rust-lang.org/std/collections/struct.HashMap.html
