# Scope and Privacy

We're going to convert all our `Interval` related code into a module. To define
a module, we use the `mod` keyword followed by the module's name and enclose the
body within curly braces.

```rust,noplayground
mod interval {
    // module body
}
```

Here's the new version of our rustle program with the `Interval` related parts
moved inside the `interval` module. In addition, comments have been added to the
module which will be used to generate documentation in an upcoming section. Go
ahead and run the code to see it in action!

````rust
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
# fn create_intervals(
#     lines: Vec<usize>,
#     before_context: usize,
#     after_context: usize,
# ) -> Result<Vec<Interval>, IntervalError> {
#     lines
#         .iter()
#         .map(|line| {
#             let start = line.saturating_sub(before_context);
#             let end = line.saturating_add(after_context);
#             Interval::new(start, end)
#         })
#         .collect()
# }
#
# fn merge_intervals(intervals: Vec<Interval>) -> Vec<Interval> {
#     // merge overlapping intervals
#     intervals
#         .into_iter()
#         .coalesce(|p, c| p.merge(&c).map_err(|_| (p, c)))
#         .collect()
# }
#
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
# fn main() {
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
#     // create intervals of the form [a,b] with the before/after context
#     let intervals =
#         match create_intervals(match_lines, before_context, after_context) {
#             Ok(intervals) => intervals,
#             Err(_) => {
#                 eprintln!("An error occurred while creating intervals");
#                 exit(1);
#             }
#         };
#
#     // merge overlapping intervals
#     let intervals = merge_intervals(intervals);
#
#     // print the lines
#     print_results(intervals, lines);
# }
#
mod interval {
    /// A list specifying general categories of Interval errors.
    enum IntervalError {
        /// Start is not less than or equal to end
        StartEndRangeInvalid,
        /// Two intervals to be merged do not overlap
        NonOverlappingInterval,
    }

    /// A closed-interval [`start`, `end`] type used for representing a range of
    /// values between `start` and `end` inclusively.
    ///
    /// # Examples
    ///
    /// You can create an `Interval` using `new`.
    ///
    /// ```rust
    /// let interval = Interval::new(1, 10).unwrap();
    /// assert_eq!(interval.start, 1);
    /// assert_eq!(interval.end, 10);
    /// ```
    struct Interval {
        start: usize,
        end: usize,
    }

    impl Interval {
        /// Creates a new `Interval` set to `start` and `end`.
        ///
        /// # Examples
        ///
        /// ```rust
        /// let interval = Interval::new(1, 10).unwrap();
        /// assert_eq!(interval.start, 1);
        /// assert_eq!(interval.end, 10);
        /// ```
        fn new(start: usize, end: usize) -> Result<Self, IntervalError> {
            if start <= end {
                Ok(Self { start, end })
            } else {
                Err(IntervalError::StartEndRangeInvalid)
            }
        }

        /// Checks if two intervals overlap. Overlapping intervals have at least
        /// one point in common.
        ///
        /// # Examples
        ///
        /// ```rust
        /// let a = Interval::new(1, 3).unwrap();
        /// let b = Interval::new(3, 5).unwrap();
        /// assert_eq!(a.overlaps(&b), true);
        /// assert_eq!(b.overlaps(&a), true);
        /// ```
        ///
        /// ```rust
        /// let a = Interval::new(1, 5).unwrap();
        /// let b = Interval::new(2, 4).unwrap();
        /// assert_eq!(a.overlaps(&b), true);
        /// assert_eq!(b.overlaps(&a), true);
        /// ```
        ///
        /// ```rust
        /// let a = Interval::new(1, 3).unwrap();
        /// let b = Interval::new(4, 6).unwrap();
        /// assert_eq!(a.overlaps(&b), false);
        /// assert_eq!(b.overlaps(&a), true);
        /// ```
        fn overlaps(&self, other: &Interval) -> bool {
            self.end >= other.start
        }

        /// Merges two intervals returning a new `Interval`.
        ///
        /// The merged `Interval` range includes the union of ranges from each
        /// `Interval`.
        ///
        /// # Examples
        ///
        /// ```rust
        /// let a = Interval::new(1, 3).unwrap();
        /// let b = Interval::new(3, 5).unwrap();
        /// let c = a.merge(&b).unwrap();
        /// assert_eq!(c.start, 1);
        /// assert_eq!(c.end, 5);
        /// ```
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
}
````

Uh-oh! Looks like we have all kinds of compiler errors! Among the chaos is two
different errors having to do with scope and privacy.

- [E0412]: cannot find type `Interval` in this scope
- [E0433]: failed to resolve: use of undeclared type `Interval`

> Remember you can always use `rustc --explain EXXXX` which provides a detailed
> explanation of an error message.

## Scope

The first error indicates that the `Interval` type is not in scope. This is
because it is now part of the `interval` module. To access it, we can use the
full path name `interval::Interval` or create a shortcut with `use`. Since we
don't want to replace every occurrence of `Interval` in our code with
`interval::Interval`, we'll take advantage of `use`. Additionally, we need
access to `IntervalError`, so we'll bring that into scope at the same time. When
bringing multiple types into scope, we can wrap them in `{}` and separate them
with `,`.

With that change in place, run the code again.

````rust
# #![allow(unused_imports)]
# extern crate regex; // this is needed for the playground
use interval::{Interval, IntervalError};
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
# fn create_intervals(
#     lines: Vec<usize>,
#     before_context: usize,
#     after_context: usize,
# ) -> Result<Vec<Interval>, IntervalError> {
#     lines
#         .iter()
#         .map(|line| {
#             let start = line.saturating_sub(before_context);
#             let end = line.saturating_add(after_context);
#             Interval::new(start, end)
#         })
#         .collect()
# }
#
# fn merge_intervals(intervals: Vec<Interval>) -> Vec<Interval> {
#     // merge overlapping intervals
#     intervals
#         .into_iter()
#         .coalesce(|p, c| p.merge(&c).map_err(|_| (p, c)))
#         .collect()
# }
#
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
# fn main() {
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
#     // create intervals of the form [a,b] with the before/after context
#     let intervals =
#         match create_intervals(match_lines, before_context, after_context) {
#             Ok(intervals) => intervals,
#             Err(_) => {
#                 eprintln!("An error occurred while creating intervals");
#                 exit(1);
#             }
#         };
#
#     // merge overlapping intervals
#     let intervals = merge_intervals(intervals);
#
#     // print the lines
#     print_results(intervals, lines);
# }
#
# mod interval {
#     /// A list specifying general categories of Interval errors.
#     enum IntervalError {
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
#     struct Interval {
#         start: usize,
#         end: usize,
#     }
#
#     impl Interval {
#         /// Creates a new `Interval` set to `start` and `end`.
#         ///
#         /// # Examples
#         ///
#         /// ```rust
#         /// let interval = Interval::new(1, 10).unwrap();
#         /// assert_eq!(interval.start, 1);
#         /// assert_eq!(interval.end, 10);
#         /// ```
#         fn new(start: usize, end: usize) -> Result<Self, IntervalError> {
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
#         fn overlaps(&self, other: &Interval) -> bool {
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
#         fn merge(&self, other: &Self) -> Result<Self, IntervalError> {
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

Bugger! More compiler errors!

## Privacy

Before we defined the `interval` module, all the types and their methods were
public. However, once we enclosed everything in a module, they became private!
By default, code within a module is private from its parent modules. To make a
module public, we need to declare it with `pub mod` instead of just `mod`. Let's
add the `pub` prefix and run the program again.

````rust
# #![allow(unused_imports)]
# extern crate regex; // this is needed for the playground
# use interval::{Interval, IntervalError};
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
# fn create_intervals(
#     lines: Vec<usize>,
#     before_context: usize,
#     after_context: usize,
# ) -> Result<Vec<Interval>, IntervalError> {
#     lines
#         .iter()
#         .map(|line| {
#             let start = line.saturating_sub(before_context);
#             let end = line.saturating_add(after_context);
#             Interval::new(start, end)
#         })
#         .collect()
# }
#
# fn merge_intervals(intervals: Vec<Interval>) -> Vec<Interval> {
#     // merge overlapping intervals
#     intervals
#         .into_iter()
#         .coalesce(|p, c| p.merge(&c).map_err(|_| (p, c)))
#         .collect()
# }
#
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
# fn main() {
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
#     // create intervals of the form [a,b] with the before/after context
#     let intervals =
#         match create_intervals(match_lines, before_context, after_context) {
#             Ok(intervals) => intervals,
#             Err(_) => {
#                 eprintln!("An error occurred while creating intervals");
#                 exit(1);
#             }
#         };
#
#     // merge overlapping intervals
#     let intervals = merge_intervals(intervals);
#
#     // print the lines
#     print_results(intervals, lines);
# }
#
pub mod interval {
#     /// A list specifying general categories of Interval errors.
#     enum IntervalError {
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
#     struct Interval {
#         start: usize,
#         end: usize,
#     }
#
#     impl Interval {
#         /// Creates a new `Interval` set to `start` and `end`.
#         ///
#         /// # Examples
#         ///
#         /// ```rust
#         /// let interval = Interval::new(1, 10).unwrap();
#         /// assert_eq!(interval.start, 1);
#         /// assert_eq!(interval.end, 10);
#         /// ```
#         fn new(start: usize, end: usize) -> Result<Self, IntervalError> {
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
#         fn overlaps(&self, other: &Interval) -> bool {
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
#         fn merge(&self, other: &Self) -> Result<Self, IntervalError> {
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

More compiler errors! Who would have thought creating a module could be this
complicated? The reason for this series of errors is to help us understand
module scope and privacy. The emerging pattern is that modules default
everything to private. Simply declaring the module public isn't enough; every
type and method within the module must also be explicitly declared public if
that is the intended design. So, let's prefix the methods and types with `pub`
and hopefully achieve a successful compile!

````rust
# #![allow(unused_imports)]
# extern crate regex; // this is needed for the playground
# use interval::{Interval, IntervalError};
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
# fn create_intervals(
#     lines: Vec<usize>,
#     before_context: usize,
#     after_context: usize,
# ) -> Result<Vec<Interval>, IntervalError> {
#     lines
#         .iter()
#         .map(|line| {
#             let start = line.saturating_sub(before_context);
#             let end = line.saturating_add(after_context);
#             Interval::new(start, end)
#         })
#         .collect()
# }
#
# fn merge_intervals(intervals: Vec<Interval>) -> Vec<Interval> {
#     // merge overlapping intervals
#     intervals
#         .into_iter()
#         .coalesce(|p, c| p.merge(&c).map_err(|_| (p, c)))
#         .collect()
# }
#
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
# fn main() {
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
#     // create intervals of the form [a,b] with the before/after context
#     let intervals =
#         match create_intervals(match_lines, before_context, after_context) {
#             Ok(intervals) => intervals,
#             Err(_) => {
#                 eprintln!("An error occurred while creating intervals");
#                 exit(1);
#             }
#         };
#
#     // merge overlapping intervals
#     let intervals = merge_intervals(intervals);
#
#     // print the lines
#     print_results(intervals, lines);
# }
#
pub mod interval {
#     /// A list specifying general categories of Interval errors.
    pub enum IntervalError {
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
    pub struct Interval {
#         start: usize,
#         end: usize,
#     }
#
#     impl Interval {
#         /// Creates a new `Interval` set to `start` and `end`.
#         ///
#         /// # Examples
#         ///
#         /// ```rust
#         /// let interval = Interval::new(1, 10).unwrap();
#         /// assert_eq!(interval.start, 1);
#         /// assert_eq!(interval.end, 10);
#         /// ```
        pub fn new(start: usize, end: usize) -> Result<Self, IntervalError> {
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
        pub fn overlaps(&self, other: &Interval) -> bool {
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
        pub fn merge(&self, other: &Self) -> Result<Self, IntervalError> {
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

Alright, last time, I promise! As a final note on privacy, and as the compiler
error pointed out, even the fields of a `struct` are private by default. So, the
last step is to make the fields public. Let's do that and finally achieve a
successful compile. Fingers crossed!

````rust
# #![allow(unused_imports)]
# extern crate regex; // this is needed for the playground
# use interval::{Interval, IntervalError};
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
# fn create_intervals(
#     lines: Vec<usize>,
#     before_context: usize,
#     after_context: usize,
# ) -> Result<Vec<Interval>, IntervalError> {
#     lines
#         .iter()
#         .map(|line| {
#             let start = line.saturating_sub(before_context);
#             let end = line.saturating_add(after_context);
#             Interval::new(start, end)
#         })
#         .collect()
# }
#
# fn merge_intervals(intervals: Vec<Interval>) -> Vec<Interval> {
#     // merge overlapping intervals
#     intervals
#         .into_iter()
#         .coalesce(|p, c| p.merge(&c).map_err(|_| (p, c)))
#         .collect()
# }
#
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
# fn main() {
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
#     // create intervals of the form [a,b] with the before/after context
#     let intervals =
#         match create_intervals(match_lines, before_context, after_context) {
#             Ok(intervals) => intervals,
#             Err(_) => {
#                 eprintln!("An error occurred while creating intervals");
#                 exit(1);
#             }
#         };
#
#     // merge overlapping intervals
#     let intervals = merge_intervals(intervals);
#
#     // print the lines
#     print_results(intervals, lines);
# }
#
# pub mod interval {
#     /// A list specifying general categories of Interval errors.
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
#     pub struct Interval {
        pub start: usize,
        pub end: usize,
#     }
#
#     impl Interval {
#         /// Creates a new `Interval` set to `start` and `end`.
#         ///
#         /// # Examples
#         ///
#         /// ```rust
#         /// let interval = Interval::new(1, 10).unwrap();
#         /// assert_eq!(interval.start, 1);
#         /// assert_eq!(interval.end, 10);
#         /// ```
#         pub fn new(start: usize, end: usize) -> Result<Self, IntervalError> {
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
#         pub fn overlaps(&self, other: &Interval) -> bool {
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

Finally, the code compiles! Now, you might be wondering why we didn't have to
declare the `enum` variants as public. If you are, good catch! The reason is
that there are two exceptions to Rust's _everything is private_[^1] behavior:

1. Associated items in a `pub` Trait are public by default.
1. `enum` variants in a `pub enum` are public by default.

# Summary

This section focused heavily on emphasizing Rust's _everything is private_
default behavior when it comes to modules. As you develop software, use privacy
to your advantage and carefully decide what parts of the API should be made
public.

Rust goes into great detail with regard to project management. As you create
your own modules and crates, reviewing the section on [Managing Growing Projects
with Packages, Crates, and Modules] will be extremely valuable.

# Exercises

> These exercises must be completed locally.

- Move the `interval` module out of `main.rs` and into a separate file. Explore
  two approaches:
  1. First, create a file `interval.rs` in the `src` directory.
  1. Next, create a directory under `src` called `interval` and move
     `interval.rs` inside.
- **Advanced**: Creating an external crate
  1. Create a library crate `cargo new --lib interval` and move the interval
     code into it.
  1. If you haven't already, create a binary crate `cargo new rustle` and update
     the `Cargo.toml` file to use the external crate from the previous step.
     Refer to the section on [Specifying Dependencies] in [The Cargo Book] for
     guidance.

# Next

Now, let's dive into generic types and make our `Interval` generic!

[Visibility and Privacy]:
  https://doc.rust-lang.org/reference/visibility-and-privacy.html
[Managing Growing Projects with Packages, Crates, and Modules]:
  https://doc.rust-lang.org/book/ch07-00-managing-growing-projects-with-packages-crates-and-modules.html
[The Cargo Book]: https://doc.rust-lang.org/cargo/
[Specifying Dependencies]:
  https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html

---

[^1]: Refer to the [Visibility and Privacy] reference for details.
