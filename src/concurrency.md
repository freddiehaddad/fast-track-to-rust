# Concurrency

We're nearing the end of the course, and there's no better way to wrap things up
than with concurrency! Rust, like many other languages, includes a native
[thread] module for spawning threads and performing tasks in parallel. However,
numerous crates build on Rust's native multithreading capabilities, simplifying
the process and offering various threading patterns.

For instance, if you've programmed in Go, you're likely familiar with channels
for moving data between `goroutines`. Similarly, if you've worked with
JavaScript, you probably know about `async` and `await`. Both of these
paradigms, along with many others, are available as crates in Rust.

Here's a few examples worth exploring:

- **[crossbeam]**: A set of tools for concurrent programming
- **[rayon]**: A data-parallelism library for Rust.
- **[tokio]**: An event-driven, non-blocking I/O platform for writing
  asynchronous I/O backed applications.

Let's dive into multithreading!

[crossbeam]: https://crates.io/crates/crossbeam
[rayon]: https://crates.io/crates/rayon
[thread]: https://doc.rust-lang.org/std/thread/
[tokio]: https://crates.io/crates/tokio
