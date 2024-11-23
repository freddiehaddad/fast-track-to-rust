# Cursor

Our production grep program now has the capability to access real files.
However, the Rust Playground does not support opening files directly. To ensure
this course remains functional in your web browser, we need to use an in-memory
buffer to simulate a file. This technique of mocking an open file is also
commonly used in unit testing, making it a valuable concept to explore.

The [`Cursor`] [^1] is used with in-memory buffers to provide [`Read`] or
[`Write`] functionality. Without digressing too much, the `BufReader` we are
using works on objects that implement the [`Read`] trait. For now, think of a
[trait] as an interface in object-oriented programming languages, with some
differences.

## Mocking a File with Cursor

We only need to make a few changes to our program to utilize `Cursor`. Let's
start by updating the `read_file` function to accept any object that implements
the `Read` trait as an argument.

```rust,noplayground
fn read_file(file: impl Read) -> Vec<String>
```

Next, we'll reintroduce our famous poem and use it as our in-memory buffer to
represent our file.

```rust,noplayground
let poem = "I have a little shadow that goes in and out with me,
            And what can be the use of him is more than I can see.
            He is very, very like me from the heels up to the head;
            And I see him jump before me, when I jump into my bed.

            The funniest thing about him is the way he likes to grow -
            Not at all like proper children, which is always very slow;
            For he sometimes shoots up taller like an india-rubber ball,
            And he sometimes gets so little that there’s none of him at all.";

let mock_file = std::io::Cursor::new(poem);
```

Finally, we comment out the lines that handled opening a file and calling
`read_file`, and instead, we directly call `read_file` with `mock_file`.

```rust,noplayground
let lines = read_file(mock_file);
// // attempt to open the file
// let lines = match File::open(filename) {
//     Ok(file) => read_file(file),
//     Err(e) => {
//         eprintln!("Error opening {filename}: {e}");
//         exit(1);
//     }
// };
```

## Putting it All Together

With these changes applied to our grep program, we can once again utilize the
Rust Playground to extend its functionality and continue learning Rust.

```rust
# use std::fs::File;
use std::io::{BufRead, BufReader, Read};
# use std::process::exit;

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
#                 line.saturating_add(after_context),
#             )
#         })
#         .collect()
# }
#
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
#
#     let pattern = "all";
#     let before_context = 1;
#     let after_context = 1;

    // // attempt to open the file
    // let lines = match File::open(filename) {
    //     Ok(file) => read_file(file),
    //     Err(e) => {
    //         eprintln!("Error opening {filename}: {e}");
    //         exit(1);
    //     }
    // };
#
#     // store the 0-based line number for any matched line
#     let match_lines: Vec<_> = lines
#         .iter()
#         .enumerate()
#         .filter_map(|(i, line)| match line.contains(pattern) {
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
#     let mut intervals =
#         create_intervals(before_context, after_context, match_lines, &lines);
#
#     // merge overlapping intervals
#     intervals.dedup_by(|next, prev| {
#         if prev.1 < next.0 {
#             false
#         } else {
#             prev.1 = next.1;
#             true
#         }
#     });
#
#     // print the lines
#     for (start, end) in intervals {
#         for (line_no, line) in
#             lines.iter().enumerate().take(end + 1).skip(start)
#         {
#             println!("{}: {}", line_no + 1, line)
#         }
#     }
}
```

What? You don't believe me! Give it a whirl and see for yourself! 😄

# Next

We're ready to add support for command line arguments and regular expressions
for pattern matching. We'll take a brief detour to learn about project
management in Rust, which will allow us to use packages (also known as crates)
to add that functionality.

[`Cursor`]: https://doc.rust-lang.org/std/io/struct.Cursor.html
[`Seek`]: https://doc.rust-lang.org/std/io/trait.Seek.html
[`Read`]: https://doc.rust-lang.org/std/io/trait.Read.html
[`Write`]: https://doc.rust-lang.org/std/io/trait.Write.html
[trait]: https://doc.rust-lang.org/book/ch10-02-traits.html

---

[^1]:
    A Cursor wraps an in-memory buffer and provides it with a [`Seek`]
    implementation.
