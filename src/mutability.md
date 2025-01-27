# Mutability

Two lines of code were added to our rustle program on lines 13 and 18. The
variable `i` will get incremented with each loop iteration and we'll use it to
print the line number. Go ahead and run the code.

```rust,editable
fn main() {
    let pattern = "him";
    let poem = "I have a little shadow that goes in and out with me,
                And what can be the use of him is more than I can see.
                He is very, very like me from the heels up to the head;
                And I see him jump before me, when I jump into my bed.

                The funniest thing about him is the way he likes to grow -
                Not at all like proper children, which is always very slow;
                For he sometimes shoots up taller like an india-rubber ball,
                And he sometimes gets so little that there's none of him at all.";

    let i = 1;
    for line in poem.lines() {
        if line.contains(pattern) {
            println!("{i}: {line}");
        }
        i += 1;
    }
}
```

Uh-oh! Something went wrong. Luckily, the compiler (`rustc`) emits very helpful
error messages.[^1] The variable `i` is immutable!

## Variables and Mutability

Variables in Rust are immutable by default. This design choice helps you write
code that leverages Rust's memory safety and fearless concurrency features. For
more details, you can refer to the section on [variables and mutability] in the
Rust documentation. While the language encourages immutability, there are
situations where mutating values is necessary. In such cases, we use the `mut`
keyword.

```rust,noplayground
let mut i = 1;
```

Go ahead and add the `mut` keyword to line 13 and run the program again.

We're going to explore cases where immutability is needed and appropriate.
However, let's see how iterators can be used to avoid the need for a mutable
counter.

______________________________________________________________________

[^1]: A lot of effort has been put into making `rustc` have great error messages.
    To help understand how to interpret them, refer to the
    [diagnostic structure] in the documentation.

[diagnostic structure]: https://rustc-dev-guide.rust-lang.org/diagnostics.html#diagnostic-structure
[variables and mutability]: https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html
