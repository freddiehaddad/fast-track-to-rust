# Variables

Rust is a statically typed language, meaning that all variables have a type, and
this type must be known at compile time. The `let` keyword is used to declare
variables, or more precisely, for variable binding.

The anatomy of a `let` statement[^1] [^2]:

```rust,noplayground
let identifier: type = expression;
```

## Type Inference

Since `y` is passed as an argument to the `print_value` function, which requires
a signed 32-bit integer, the compiler infers its type. Therefore, the explicit
type declaration for `x` can be omitted.

```rust,editable
fn print_value(value: i32) {
    println!("{value}");
}

fn main() {
    let x: i32 = 10;
    let y = 20;

    print_value(x);
    print_value(y);
}
```

## Rustle Variables

To begin with our rustle program, we'll avoid handling user input via command
line arguments for now. Instead, we'll hard code some strings and perform some
simple _rustling_. Let's use the famous poem _My Shadow_ by the poet Robert
Louis Stevenson as our input.

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
}
```

The next step is to search the poem for occurrences of the pattern and print the
results. To achieve this, we'll need to learn a bit about control flow.

# Summary

In this section, we:

- Learned how to declare variables.
- Explored Rust's type inference.

# Next

Onward to control flow.

[^1]: [`let` statements] support more advanced features that are not being covered
    yet.

[^2]: In many cases, the compiler can infer the type allowing you to omit it.

[`let` statements]: https://doc.rust-lang.org/reference/statements.html#let-statements
