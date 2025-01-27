# Non-scoped Threads

The simplest way to enable multithreading support in our rustle program is to
spawn a thread for each specified file, perform the pattern matching steps we've
developed, and print the results using a fork/join pattern. If you have a C++
background, you might start by exploring the functions in the [thread] module,
discovering the [spawn] function, and diving right in.

However, this approach is flawed! While it might be a valuable learning
experience, it will likely be time-consuming, frustrating, and confusing. Let's
examine some of the pitfalls of this approach and then explore a more effective
solution.

## The `spawn` Function

Here's the signature for the `spawn` function:

```rust,noplayground
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
where
    F: FnOnce() -> T + Send + 'static,
    T: Send + 'static,
```

The `spawn` function takes a closure (`f`) as an argument, spawns a new thread
to run the closure, and returns a [`JoinHandle`]. As the name suggests, the
`JoinHandle` provides a [`join`] method to wait for the spawned thread to
finish. So far, so good. However, the `'static`[^1] constraint on `F` and the
return value `T` is where, as the Bard would say, lies the rub.

The `'static` constraint requires that the closure and its return value must
live for the entire duration of the program's execution. This is necessary
because threads can outlive the context in which they were created.

If a thread, and by extension its return value, can outlive its caller, we need
to ensure they remain valid afterward. Since we _can't_ predict when the thread
will finish, they must be valid until the end of the program. Hence, the
`'static` lifetime.

> Don't worry if this concept is still unclear; it's one of the most challenging
> aspects of Rust, and many developers find it difficult at first.

Let's examine a simple program to better understand these concepts. Review the
following code snippet and then run the program.

```rust
fn main() {
    use std::thread;

    let error = 0373; // Compiler error E0373

    let handler = thread::spawn(|| {
        println!("{error}");
    });

    handler.join().unwrap();
}
```

This simple program seems perfectly valid. So why does the Rust compiler
complain about code that would run without issues in most other languages? Even
though it's not the case in this example, in a more general sense, the spawned
thread created in `main` could potentially outlive the thread from which it was
created. Because the closure borrows the `error` value, the parent thread could
go out of scope, causing the `error` value to be dropped rendering the reference
invalid. Because the compiler _can't_ know whether or not that's going to
happen, we see the error:

> error[E0373]: closure may outlive the current function, but it borrows
> `error`, which is owned by the current function

The compiler does offer a solution that may work under some circumstances:

> help: to force the closure to take ownership of `error` (and any other
> referenced variables), use the [`move`] [^2] keyword

Let's see what happens if we do that:

```rust
fn main() {
    use std::thread;

    let error = 0373; // Compiler error E0373

    let handler = thread::spawn(move || {
        println!("{error}");
    });

    handler.join().unwrap();
}
```

Recalling our discussion on _copy semantics_, it probably makes sense why this
worked. The value was _copied_ into the scope of the spawned thread, making it
independent from the original and solving the lifetime problem. The variable is
now guaranteed to live for as long as the thread. This works fine for primitive
types and types implementing the `Copy` trait. But what if copying is an
expensive operation, or the object can't be copied? This leads to the next
challenge that developers learning Rust often face: what happens if we need to
spawn additional threads that also need access to the same variable?

For example, consider this case:

```rust
fn main() {
    use std::thread;

    let error = String::from("E0373"); // Compiler error E0373

    let handler1 = thread::spawn(move || {
        println!("{error}");
    });

    let handler2 = thread::spawn(move || {
        println!("{error}");
    });

    handler1.join().unwrap();
    handler2.join().unwrap();
}
```

While these strict rules may seem daunting, especially in larger programs, they
are the foundation of Rust's [fearless concurrency] goals. These rules ensure
that concurrency in your programs is both safe and efficient.

Now that we understand why the traditional fork/join model, which works in many
other languages, is likely to fail in Rust, let's explore how to correctly
implement this approach!

______________________________________________________________________

[^1]: Lifetimes are another type of generic. However, instead of ensuring that a
    type has the desired behavior, they ensure that references remain valid for
    the desired duration.

[^2]: `move` converts any variables captured by reference or mutable reference to
    variables captured by value.

[e0373]: https://doc.rust-lang.org/stable/error_codes/E0373.html
[fearless concurrency]: https://doc.rust-lang.org/book/ch16-00-concurrency.html#fearless-concurrency
[spawn]: https://doc.rust-lang.org/stable/std/thread/fn.spawn.html
[thread]: https://doc.rust-lang.org/std/thread/
[`joinhandle`]: https://doc.rust-lang.org/stable/std/thread/struct.JoinHandle.html
[`join`]: https://doc.rust-lang.org/stable/std/thread/struct.JoinHandle.html#method.join
[`move`]: https://doc.rust-lang.org/std/keyword.move.html
