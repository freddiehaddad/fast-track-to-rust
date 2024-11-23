# Interval

The current version of our `Interval` is limited to the `usize` type. If we want
to support floating point values, negative integers, or other exotic numeric
types, it won't work. We're going to address this by making it compatible with
any numeric type, with some restrictions.[^1]

## Generic Data Type

The current definition of our `Interval` `struct` specifies the field types as
`usize`. To redefine the `struct` to use a generic type parameter for its
fields, we use the `<>` syntax.

```rust,noplayground
pub struct Interval<T> {
    pub start: T,
    pub end: T,
}
```

> By using a single generic type to define `Interval<T>`, we indicate that the
> `Interval<T>` `struct` is generic over some type `T`, and the fields `start`
> and `end` are both of that same type, whatever it may be. If we attempt to
> create an instance of `Interval<T>` with fields of different types, our code
> won't compile. To support different types for the fields, we would need to
> specify additional generic types, such as `struct Interval<T, U>`.

You've probably guessed it already, but this code won't compile. Take a moment
to think about why that might be, and then run the code to see what the compiler
has to say.

````rust
# #![allow(unused_imports)]
# #![allow(dead_code)]
# extern crate itertools; // this is needed for the playground
# extern crate regex; // this is needed for the playground
# use interval::{Interval, IntervalError};
# use itertools::Itertools;
# use regex::Regex;
# use std::fs::File;
# use std::io::{BufRead, BufReader, Read};
# use std::process::exit;
#
# fn create_intervals(
#     before_context: usize,
#     after_context: usize,
#     match_lines: Vec<usize>,
#     lines: &[String],
# ) -> Result<Vec<Interval>, IntervalError> {
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
# fn merge_intervals(intervals: Vec<Interval>) -> Vec<Interval> {
#     intervals
#         .into_iter()
#         .coalesce(|p, c| p.merge(&c).or(Err((p, c))))
#         .collect()
# }
#
# fn print_matches(intervals: Vec<Interval>, lines: &[String]) {
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
#
#     print_matches(intervals, &lines);
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
    pub struct Interval<T> {
        pub start: T,
        pub end: T,
    }
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

As you guessed it, there's a lot of compiler errors and they're very helpful!

- [E0107]: missing generics for `struct` `Interval`

Let's walk through the errors one by one. The first error informs us that the
return value on line 17 references a generic type but fails to specify one. It
shows the definition of the type on line 144 and provides a helpful tip for how
to correct it. The compiler suggests appending `<T>` to `Interval` and
specifying the type for `T`, which in our case will be `usize`.

```console
error[E0107]: missing generics for struct `Interval`
   --> src/main.rs:17:17
    |
17  | ) -> Result<Vec<Interval>, IntervalError> {
    |                 ^^^^^^^^ expected 1 generic argument
    |
note: struct defined here, with 1 generic parameter: `T`
   --> src/main.rs:144:16
    |
144 |     pub struct Interval<T> {
    |                ^^^^^^^^ -
help: add missing generic argument
    |
17  | ) -> Result<Vec<Interval<T>>, IntervalError> {
    |                         +++
```

The fix should be relatively straightforward to understand, as we've used the
same approach for other generic types we've worked with. Let's apply the fix and
try again!

````rust
# #![allow(unused_imports)]
# #![allow(dead_code)]
# extern crate itertools; // this is needed for the playground
# extern crate regex; // this is needed for the playground
# use interval::{Interval, IntervalError};
# use itertools::Itertools;
# use regex::Regex;
# use std::fs::File;
# use std::io::{BufRead, BufReader, Read};
# use std::process::exit;
#
fn create_intervals(
    before_context: usize,
    after_context: usize,
    match_lines: Vec<usize>,
    lines: &[String],
) -> Result<Vec<Interval<usize>>, IntervalError> {
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
fn merge_intervals(intervals: Vec<Interval<usize>>) -> Vec<Interval<usize>> {
#     intervals
#         .into_iter()
#         .coalesce(|p, c| p.merge(&c).or(Err((p, c))))
#         .collect()
# }
#
fn print_matches(intervals: Vec<Interval<usize>>, lines: &[String]) {
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
#
#     print_matches(intervals, &lines);
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
#     pub struct Interval<T> {
#         pub start: T,
#         pub end: T,
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

As expected, we still have some more compiler errors to work through and these
lead us into how we implement methods for a generic type!

```console
error[E0107]: missing generics for struct `Interval`
   --> src/main.rs:149:10
    |
149 |     impl Interval {
    |          ^^^^^^^^ expected 1 generic argument
    |
note: struct defined here, with 1 generic parameter: `T`
   --> src/main.rs:144:16
    |
144 |     pub struct Interval<T> {
    |                ^^^^^^^^ -
help: add missing generic argument
    |
149 |     impl Interval<T> {
    |                  +++

error[E0107]: missing generics for struct `Interval`
   --> src/main.rs:192:40
    |
192 |         pub fn overlaps(&self, other: &Interval) -> bool {
    |                                        ^^^^^^^^ expected 1 generic argument
    |
note: struct defined here, with 1 generic parameter: `T`
   --> src/main.rs:144:16
    |
144 |     pub struct Interval<T> {
    |                ^^^^^^^^ -
help: add missing generic argument
    |
192 |         pub fn overlaps(&self, other: &Interval<T>) -> bool {
    |                                                +++
```

## Inherent Implementation

There are two main types of implementation we define: inherent implementations
(standalone) and trait implementations.[^2] We'll focus on inherent
implementation first and explore trait implementations towards the end of this
section.

We need to update our `impl` block to support generic type parameters for our
generic type, and the syntax is as follows:

```rust,noplayground
impl<T> Type<T> {}  // Type is replaced with Interval for our case
```

Here are the necessary changes. Review them and run the program again.

````rust
# #![allow(unused_imports)]
# #![allow(dead_code)]
# extern crate itertools; // this is needed for the playground
# extern crate regex; // this is needed for the playground
# use interval::{Interval, IntervalError};
# use itertools::Itertools;
# use regex::Regex;
# use std::fs::File;
# use std::io::{BufRead, BufReader, Read};
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
#
# fn print_matches(intervals: Vec<Interval<usize>>, lines: &[String]) {
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
#
#     print_matches(intervals, &lines);
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
#     pub struct Interval<T> {
#         pub start: T,
#         pub end: T,
#     }
#
    impl<T> Interval<T> {
#         /// Creates a new `Interval` set to `start` and `end`.
#         ///
#         /// # Examples
#         ///
#         /// ```rust
#         /// let interval = Interval::new(1, 10).unwrap();
#         /// assert_eq!(interval.start, 1);
#         /// assert_eq!(interval.end, 10);
#         /// ```
        pub fn new(start: T, end: T) -> Result<Self, IntervalError> {
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
        pub fn overlaps(&self, other: &Interval<T>) -> bool {
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

Well, that didn't work! Now we have two new compiler errors:

- [E0369]: binary operation `<=` cannot be applied to type `T`
- [E0507]: cannot move out of `self.start` which is behind a shared reference

That's okay, because these errors are a perfect segue into trait bounds!

## Trait Bounds

The compiler generated a few very helpful error messages, which we can
categorize into two types, both of which suggest the use of trait bounds:

```console
error[E0369]: binary operation `<=` cannot be applied to type `T`
   --> src/main.rs:160:22
    |
160 |             if start <= end {
    |                ----- ^^ --- T
    |                |
    |                T
    |
help: consider restricting type parameter `T`
    |
149 |     impl<T: std::cmp::PartialOrd> Interval<T> {
    |           ++++++++++++++++++++++

error[E0507]: cannot move out of `self.start` which is behind a shared reference
   --> src/main.rs:213:28
    |
213 |                     start: self.start,
    |                            ^^^^^^^^^^ move occurs because `self.start` has type
    |                                       `T`, which does not implement the `Copy`
    |                                       trait
    |
```

With an introduction to trait bounds, everything will become clear. Let's begin
with the first error. The compiler indicates that a comparison between two
generic types is being attempted in the `new` function, and not all types
support this capability. In other words, not all types implement the
[`PartialOrd`] trait, which is necessary for using comparison operators like
`<=` and `>=`.

> For those with an object-oriented programming background, a trait can be
> thought of as similar to an interface.

```rust,noplayground
pub fn new(start: T, end: T) -> Result<Self, IntervalError> {
    if start <= end {
        Ok(Self { start, end })
    } else {
        Err(IntervalError::StartEndRangeInvalid)
    }
}
```

Since our `Interval` `struct` is generic over `T`, the types for `start` and
`end` can be anything, including types that aren't comparable. Therefore, the
compiler is instructing us to restrict `T` to any type that implements the
[`PartialOrd`] trait. In other words, we need to place a _bound_ on `T`.

The next error requires us to revisit the topic of move semantics in Rust.
Except for primitive types, most standard library types and those found in
third-party crates do not implement the [`Copy`] trait. In other words, they do
not have _copy semantics_. Let's examine the function where the error occurs:

```rust,noplayground
pub fn merge(&self, other: &Self) -> Result<Self, IntervalError> {
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

The `merge` function borrows the values for `self` and `other`. If the two
intervals overlap, a new interval is returned with the values copied into it.
Since we have been using `usize`, a type that implements the `Copy` trait, this
worked. To ensure it continues to work, we need to place another bound on `T`,
limiting it to types that implement the `Copy` trait.

> The compiler also mentions the [`Clone`] trait, which allows for explicit
> duplication of an object via the `clone` method. This means we could specify
> the trait bound as `Clone` instead of `Copy` and update the method to
> explicitly call `clone`.

With an understanding of trait bounds, let's examine how to impose these
restrictions on our `Interval`. Does the program work now?

````rust
# #![allow(unused_imports)]
# #![allow(dead_code)]
# extern crate itertools; // this is needed for the playground
# extern crate regex; // this is needed for the playground
# use interval::{Interval, IntervalError};
# use itertools::Itertools;
# use regex::Regex;
# use std::fs::File;
# use std::io::{BufRead, BufReader, Read};
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
#
# fn print_matches(intervals: Vec<Interval<usize>>, lines: &[String]) {
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
#
#     print_matches(intervals, &lines);
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
#     pub struct Interval<T> {
#         pub start: T,
#         pub end: T,
#     }
#
    impl<T: Copy + PartialOrd> Interval<T> {
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

Voila! And we now have a generic `Interval` module that can support any type
implementing the `Copy` and `PartialOrd` traits!

# Exercise

- Modify the program to use the `Clone` trait instead of `Copy`.

<details>
    <summary>Solution</summary>

```rust,noplayground
impl<T: Clone + PartialOrd> Interval<T> {
    pub fn merge(&self, other: &Self) -> Result<Self, IntervalError> {
        if self.overlaps(other) {
            Ok(Self {
                start: self.start.clone(),
                end: other.end.clone(),
            })
        } else {
            Err(IntervalError::NonOverlappingInterval)
        }
    }
}
```

</details>

````rust,editable
#![allow(unused_imports)]
#![allow(dead_code)]
extern crate itertools; // this is needed for the playground
extern crate regex; // this is needed for the playground
use interval::{Interval, IntervalError};
use itertools::Itertools;
use regex::Regex;
use std::fs::File;
use std::io::{BufRead, BufReader, Read};
use std::process::exit;

fn create_intervals(
    before_context: usize,
    after_context: usize,
    match_lines: Vec<usize>,
    lines: &[String],
) -> Result<Vec<Interval<usize>>, IntervalError> {
    match_lines
        .iter()
        .map(|line| {
            let start = line.saturating_sub(before_context);
            let end = line.saturating_add(after_context);
            Interval::new(start, end)
        })
        .collect()
}

fn merge_intervals(intervals: Vec<Interval<usize>>) -> Vec<Interval<usize>> {
    intervals
        .into_iter()
        .coalesce(|p, c| p.merge(&c).or(Err((p, c))))
        .collect()
}

fn print_matches(intervals: Vec<Interval<usize>>, lines: &[String]) {
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
    // let filename = "poem.txt";
    let poem = "I have a little shadow that goes in and out with me,
                And what can be the use of him is more than I can see.
                He is very, very like me from the heels up to the head;
                And I see him jump before me, when I jump into my bed.

                The funniest thing about him is the way he likes to grow -
                Not at all like proper children, which is always very slow;
                For he sometimes shoots up taller like an india-rubber ball,
               And he sometimes gets so little that there’s none of him at all.";

    let mock_file = std::io::Cursor::new(poem);
    let lines = read_file(mock_file);

    let pattern = "little";
    let before_context = 1;
    let after_context = 1;

    // // attempt to open the file
    // let lines = match File::open(filename) {
    //     Ok(file) => read_file(file),
    //     Err(e) => {
    //         eprintln!("Error opening {filename}: {e}");
    //         exit(1);
    //     }
    // };

    // compile the regular expression
    let regex = match Regex::new(pattern) {
        Ok(re) => re, // bind re to regex
        Err(e) => {
            eprintln!("{e}"); // write to standard error
            exit(1);
        }
    };

    // store the 0-based line number for any matched line
    let match_lines: Vec<_> = lines
        .iter()
        .enumerate()
        .filter_map(|(i, line)| match regex.is_match(line) {
            true => Some(i),
            false => None,
        })
        .collect(); // turns anything iterable into a collection

    // exit early if no matches were found
    if match_lines.is_empty() {
        return;
    }

    // create intervals of the form [a,b] with the before/after context
    let intervals = match create_intervals(
        before_context,
        after_context,
        match_lines,
        &lines,
    ) {
        Ok(intervals) => intervals,
        Err(_) => {
            eprintln!("An error occurred while creating intervals");
            exit(1);
        }
    };

    let intervals = merge_intervals(intervals);

    print_matches(intervals, &lines);
}

pub mod interval {
    /// A list specifying general categories of Interval errors.
    pub enum IntervalError {
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
    pub struct Interval<T> {
        pub start: T,
        pub end: T,
    }

    impl<T: Copy + PartialOrd> Interval<T> {
        /// Creates a new `Interval` set to `start` and `end`.
        ///
        /// # Examples
        ///
        /// ```rust
        /// let interval = Interval::new(1, 10).unwrap();
        /// assert_eq!(interval.start, 1);
        /// assert_eq!(interval.end, 10);
        /// ```
        pub fn new(start: T, end: T) -> Result<Self, IntervalError> {
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
        pub fn overlaps(&self, other: &Interval<T>) -> bool {
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
        pub fn merge(&self, other: &Self) -> Result<Self, IntervalError> {
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

# Next

Let's dive deeper into traits!

[trait bounds]: https://doc.rust-lang.org/reference/trait-bounds.html
[Rust book]:
  https://doc.rust-lang.org/book/ch10-02-traits.html#returning-types-that-implement-traits
[`PartialOrd`]: https://doc.rust-lang.org/std/cmp/trait.PartialOrd.html
[`Copy`]: https://doc.rust-lang.org/std/marker/trait.Copy.html
[`CLone`]: https://doc.rust-lang.org/std/clone/trait.Clone.html

[^1]:
    The restrictions we place on the supported types are known as [trait
    bounds].

[^2]: For more information on `impl Trait` syntax, see the [Rust book]
