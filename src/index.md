# Welcome to Fast Track to Rust

This course is designed to introduce you to the Rust programming language by
building an actual program. We'll be developing a grep-like program, called
rustle[^1] with a minimal subset of the features found in the [GNU] [^2] [grep]
[^3] utility. This means we'll start with what we know and iterate upon the
design as we learn more about the language.

> The official URL for this online course is:
> [https://freddiehaddad.github.io/fast-track-to-rust](https://freddiehaddad.github.io/fast-track-to-rust)
>
> The source code can be found at:
> [https://github.com/freddiehaddad/fast-track-to-rust](https://github.com/freddiehaddad/fast-track-to-rust)

The goal is to teach you Rust, so it's assumed you don't know anything about the
language. However, it's also assumed that you have experience programming in
other languages like C++, and are familiar with multithreading, data structures,
and program memory organization (i.e., heap, stack, etc.).

This course moves relatively quickly. Most topics are covered enough to provide
a fundamental understanding. As you progress, references to the official Rust
[documentation] are provided for further exploration.

Enough chit chat, let's get started!

![Fast Track to Rust Logo](logo.svg)

______________________________________________________________________

[^1]: The name Rustle was chosen because it's a play on the word "rust" and
    represents the sound of leaves rustling as the program searches through
    data.

[^2]: GNU is an collection of free software commonly used as an operating system.
    The family of operating systems, known as Linux, are built from them.

[^3]: Grep searches one or more input files for lines containing a match to a
    specified pattern. By default, grep outputs the matching lines.

[documentation]: https://doc.rust-lang.org/book/title-page.html
[gnu]: https://www.gnu.org/
[grep]: https://www.gnu.org/software/grep/manual/grep.html
