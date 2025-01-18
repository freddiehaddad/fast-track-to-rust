# Programming Concepts

It's time to start adding some functionality to our rustle program. In the next
few sections, we'll cover topics such as variables and mutability, iterators,
optional values, and closures. These topics are quite extensive, and covering
them in full detail is not the objective. Instead, we'll lay the foundation and
share core fundamentals to help you gain enough understanding to leverage the
documentation successfully. Using the provided links is highly recommended for
full details and comprehension.

## Rustle Features

Rustle is a fairly large program with numerous features. In this course, we
won't be implementing all of them. Instead, we'll concentrate on a subset of
features (with slight variations in behavior) that will help you learn Rust. The
features we'll be implementing are:

| command line argument            | description                                                                    |
| -------------------------------- | ------------------------------------------------------------------------------ |
| `-n`, `--line-number`            | Prefix each line of output with the 1-based line number within its input file. |
| `-A num`, `--after-context=num`  | Print _num_ lines of trailing context after matching lines.                    |
| `-B num`, `--before-context=num` | Print _num_ lines of leading context before matching lines.                    |

## Printing Line Numbers

The first feature we're going to add to our rustle program is the ability to
print the line number of any lines that match our pattern. Coming from a
language like C/C++, you may be inclined to handle this with a variable that we
increment with each loop iteration. While this would work, it introduces some of
the issues that plague languages like C and C++ (e.g., out-of-bounds indexing,
integer overflow, and more). We'll demonstrate this and follow it up with a more
idiomatic approach. Let's get going!

[crates]:
  https://doc.rust-lang.org/reference/glossary.html?highlight=crate#crate
[standard library]: https://doc.rust-lang.org/std/index.html
[traits]: https://doc.rust-lang.org/reference/items/traits.html
