# Welcome to Fast Track to Rust

This course is designed to introduce you to the Rust programming language by
building an actual program. We'll be developing a grep-like program with a
minimal subset of the features found in the [GNU] [^1] [grep] [^2] utility. This
means we'll start with what we know and iterate upon the design as we learn more
about the language.

The goal is to teach you Rust, so it's assumed you don't know anything about the
language. However, it's also assumed that you have experience programming in
other languages like C++, and are familiar with multithreading, data structures,
and program memory organization (i.e., heap, stack, etc.).

This course moves relatively quickly. Most topics are covered enough to provide
a fundamental understanding. As you progress, references to the official Rust
[documentation] are provided for further exploration.

Enough chit chat, let's get started!

![Fast Track to Rust Logo](logo.svg)

[grep]: https://www.gnu.org/software/grep/manual/grep.html
[GNU]: https://www.gnu.org/
[documentation]: https://doc.rust-lang.org/book/title-page.html

---

[^1]:
    GNU is an collection of free software commonly used as an operating system.
    The family of operating systems, known as Linux, are built from them.

[^2]:
    Grep searches one or more input files for lines containing a match to a
    specified pattern. By default, Grep outputs the matching lines.