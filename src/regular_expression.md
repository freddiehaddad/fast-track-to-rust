# Regular Expressions

With the `Regex` crate added to our project, we'll replace the `pattern` string
slice we've been using with a regular expression.

## Using Regex

The `Regex` modules defines a `new` method that takes a regular expression,
attempts to compile it, and returns a `Regex` object.[^1]

```rust,noplayground
let pattern = "[Ee]xample";
let re = Regex::new(pattern);
```

Since compiling a regular expression can fail (e.g., due to an invalid pattern),
`new` returns a `Result`. Here is the function signature for `new`[^2]:

```rust,noplayground
fn new(re: &str) -> Result<Regex, Error>
```

The function signature indicates that the `Ok` variant returns a `Regex`, while
the `Err` variant returns an `Error`. Since our grep program can't continue with
an invalid regular expression, we need to catch that case, display a helpful
error message, and exit the program.

Let's put all this together:

```rust,editable
extern crate regex; // this is needed for the playground
use regex::Regex;
use std::process::exit;


fn main() {
    let pattern = "(missing the closing parenthesis"; // invalid expression

    // compile the regular expression
    match Regex::new(pattern) {
        // the underscore (_) means we are ignoring the value returned by new
        Ok(_) => println!("{pattern} is a valid regular expression!"),

        // e is the error value returned by new
        Err(e) => {
            eprintln!("{e}"); // eprintln! writes to standard error
            exit(1);          // exit with error code 1
        }
    };
}
```

Run the code to see the error. Then, correct the it by adding the missing
parenthesis `)` and re-run the code.

## Updating Grep

We now have enough context to modify our Grep program to include regular
expression support. Below are the changes, with the unrelated parts of the
program hidden:

```rust
# extern crate regex; // this is needed for the playground
use regex::Regex;
# use std::fs::File;
# use std::io::{BufRead, BufReader, Read};
# use std::process::exit;
#
# fn create_intervals(
#     before_context: usize,
#     after_context: usize,
#     match_lines: Vec<usize>,
#     lines: &[String],
# ) -> Vec<(usize, usize)> {
#     match_lines
#         .iter()
#         .map(|line| {
#             (
#                 line.saturating_sub(before_context),
#                 (lines.len() - 1).min(line.saturating_add(after_context)),
#             )
#         })
#         .collect()
# }
#
# fn read_file(file: impl Read) -> Vec<String> {
#     BufReader::new(file).lines().map_while(Result::ok).collect()
# }

fn main() {
#     // let filename = "poem.txt";
#     let poem = "I have a little shadow that goes in and out with me,
#         And what can be the use of him is more than I can see.
#         He is very, very like me from the heels up to the head;
#         And I see him jump before me, when I jump into my bed.
#
#         The funniest thing about him is the way he likes to grow -
#         Not at all like proper children, which is always very slow;
#         For he sometimes shoots up taller like an india-rubber ball,
#         And he sometimes gets so little that there’s none of him at all.";
#
#     let mock_file = std::io::Cursor::new(poem);
#     let lines = read_file(mock_file);
#
    let pattern = "(all)|(little)";
#     let before_context = 1;
#     let after_context = 1;

#    // // attempt to open the file
#    // let lines = match File::open(filename) {
#    //     Ok(file) => read_file(file),
#    //     Err(e) => {
#    //         eprintln!("Error opening {filename}: {e}");
#    //         exit(1);
#    //     }
#    // };
#
    // compile the regular expression
    let regex = match Regex::new(pattern) {
        Ok(re) => re, // bind re to regex
        Err(e) => {
            eprintln!("{e}"); // write to standard error
            exit(1);
        }
    };
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
#     // merge overlapping intervals
#     let mut merged_intervals = vec![intervals[0]];
#     for (c_start, c_end) in intervals.iter().skip(1) {
#         let last_index = merged_intervals.len() - 1;
#         let (l_start, l_end) = merged_intervals[last_index];
#
#         if l_end < *c_start {
#             // non-overlapping case
#             merged_intervals.push((*c_start, *c_end));
#         } else {
#             // overlapping case
#             merged_intervals[last_index] = (l_start, *c_end);
#         }
#     }
#
#     // print the lines
#     for (start, end) in merged_intervals {
#         for (line_no, line) in
#             lines.iter().enumerate().take(end + 1).skip(start)
#         {
#             println!("{}: {}", line_no + 1, line)
#         }
#     }
}
```

> Don't forget, you can reveal the hidden parts by clicking _Show hidden lines_.

The `let regex = match Regex::new(pattern)` variable binding expression might
seem a bit unusual. The pattern is discussed in the Rust documentation section
on [Recoverable Errors with Result]. To briefly explain: When the result is
`Ok`, this code extracts the inner `re` value from the `Ok` variant and moves it
to the variable `regex`.

# Next

Onward to creating our own module!

[Recoverable Errors with Result]:
  https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#recoverable-errors-with-result

---

[^1]:
    The `Regex` crate includes excellent
    [documentation](https://docs.rs/regex/latest/regex/) and detailed
    [examples](https://docs.rs/regex/latest/regex/#examples) to learn from.

[^2]:
    The source code for `new` can be found
    [here](https://docs.rs/regex/latest/src/regex/regex/string.rs.html#180-182).
