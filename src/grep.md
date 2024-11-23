# Grep

Now that we've covered traits, let's ensure our projects are synchronized. Our
interval module has been enhanced to support several new capabilities. We can
print programmer-facing intervals using the `Debug` trait and user-facing
intervals using the `Display` trait. Additionally, we have limited support for
comparing intervals through the `PartialEq` and `PartialOrd` traits. Here's the
completed code:

````rust
#![allow(unused_imports)]
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
    #[derive(Debug)]
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
    #[derive(Debug, PartialEq)]
    pub struct Interval<T> {
        pub start: T,
        pub end: T,
    }

    use std::fmt;
    impl<T: fmt::Display> fmt::Display for Interval<T> {
        fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> fmt::Result {
            write!(f, "[{}, {}]", self.start, self.end)
        }
    }

    use std::cmp::Ordering;
    impl<T: PartialEq + PartialOrd> PartialOrd for Interval<T> {
        fn partial_cmp(&self, other: &Self) -> Option<Ordering> {
            if self == other {
                Some(Ordering::Equal)
            } else if self.end < other.start {
                Some(Ordering::Less)
            } else if self.start > other.end {
                Some(Ordering::Greater)
            } else {
                None // Intervals overlap
            }
        }
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

Let's shift our focus to attributes!
