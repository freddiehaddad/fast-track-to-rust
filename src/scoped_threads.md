# Scoped Threads

In the previous section, we explored the `spawn` method in the `thread` module
and noted its limitation: it cannot borrow non-`'static` data because the
compiler cannot guarantee that all threads will be joined before the lifetimes
of any borrowed values expire. To address this issue, the `thread` module also
offers the [`scope` function], which works in conjunction with [`Scope`]. This
combination allows for borrowing non-`'static` data by ensuring that all spawned
threads within the scope are automatically joined before the function in which
they were created returns, unless they were manually joined earlier.

## The `scope` Function

Here's the signature for the [`scope` function]:

```rust,noplayground
pub fn scope<'env, F, T>(f: F) -> T
where
    F: for<'scope> FnOnce(&'scope Scope<'scope, 'env>) -> T,
```

The `scope` function takes a closure (`f`) as an argument, which receives a
`Scope` object (created by `scope`). This `Scope` object allows threads to be
spawned using the [`spawn`] method. The spawn method returns a
[`ScopedJoinHandle`], which, as the name suggests, provides a [`join`] method to
wait for the spawned thread to complete.

Scoped threads involve two lifetimes: `'scope` and `'env`:

- `'env`: This is a lifetime parameter that represents the lifetime of the
  environment data that the `Scope` can borrow (meaning the data from outside
  the scope). It ensures that any references to the environment data outlive the
  scope.
- `'scope`: This is another lifetime parameter that represents the lifetime of
  the scope itself. This is the period during which new scoped threads can be
  spawned and running. It ensures that any running threads are joined before the
  lifetime ends.

In plain English, the `scope` function takes a closure `f` that can borrow data
with a specific lifetime `'env` and ensures that all threads spawned within the
scope are joined before the function returns.

Let's revisit one of the examples from the previous section that failed to
compile to better understand these concepts. Review the following code snippet
and then run the program.

```rust
fn main() {
    use std::thread;

    let error = String::from("E0373"); // Compiler error E0373

    thread::scope(|s| {
        let handler = s.spawn(|| {
            println!("{error}");
        });

        s.spawn(|| {
            println!("{error}");
        });

        handler.join().unwrap(); // Manually join the first thread

        // Second thread is automatically joined when the closure returns.
    });
}
```

Through these ownership and type system tools, it is guaranteed that all threads
created within `scope` are joined before their lifetimes end. This allows the
Rust compiler to be certain that all borrowed data (in this case, the `error`
`String`) remains valid for the lifetime of the threads. This is how Rust turns
many runtime errors into compile-time errors. Concepts like this facilitate
fearless concurrency in Rust!

[`scope` function]: https://doc.rust-lang.org/stable/std/thread/fn.scope.html
[`Scope`]: https://doc.rust-lang.org/stable/std/thread/struct.Scope.html
[`spawn`]:
  https://doc.rust-lang.org/stable/std/thread/struct.Scope.html#method.spawn
[`ScopedJoinHandle`]:
  https://doc.rust-lang.org/stable/std/thread/struct.ScopedJoinHandle.html
[`join`]:
  https://doc.rust-lang.org/stable/std/thread/struct.ScopedJoinHandle.html#method.join
