# Control Flow

Some of the common methods in Rust for controlling the flow of a program
include:

- [`if`] and [`if let`] expressions
- [Loop expressions]
- [`match` expressions]

These methods help you manage the execution flow and make your code more
efficient and readable.

We will use a `for` loop to iterate over all the lines in the poem, checking
each for the substring specified by `pattern`. Each line will be evaluated using
a `match` expression.

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
                And he sometimes gets so little that there’s none of him at all.";

    for line in poem.lines() {
        match line.contains(pattern) {
            true => println!("{line}"),
            false => (),
        }
    }
}
```

## `for` Loops

The syntax of a `for` loop should look familiar. However, something interesting
is happening on that line. Notice the method call `poem.lines()`. You might have
thought poem was a string literal. How can it have methods? Well, as you guessed
and as was mentioned earlier, string slices are more than just string literals.
We'll explore them in more detail, so keep that in mind.

The purpose of the loop is quite clear: it iterates over each line in the poem.

## `match` Expressions

You might have already figured it out, but the `match` expression is similar to
an `if` expression in that it introduces a branch in the code execution.
However, it has a very powerful feature: when used, the compiler ensures that
all possible results of the [_scrutinee_] are covered. This aspect of [`match`
expressions] guarantees that all cases are handled.

Let's ensure we fully understand this. In the code snippet, comment out line 16
by prefixing it with `//`, and then run the code.

> Friendly compiler errors
>
> Take a moment to appreciate just how helpful this compiler error is. The Rust
> compiler is truly your friend!

```text
   Compiling playground v0.0.1 (/playground)
error[E0004]: non-exhaustive patterns: `false` not covered
  --> src/main.rs:14:15
   |
14 |         match line.contains(pattern) {
   |               ^^^^^^^^^^^^^^^^^^^^^^ pattern `false` not covered
   |
   = note: the matched value is of type `bool`
help: ensure that all possible cases are being handled by adding a match arm with a
   |  wildcard pattern or an explicit pattern as shown
   |
15 ~             true => println!("{line}"),
16 ~             false => todo!(),
   |

For more information about this error, try `rustc --explain E0004`.
error: could not compile `playground` (bin "playground") due to 1 previous error
```

The compiler error message is informing you that:

- The `contains` method returns a boolean value.
- The `false` case is not covered.
- The Rust compiler can provide more information via `rustc --explain E0004`.

Let's see what the Rust compiler has to say:

```rust,noplayground
$ rustc --explain E0004
This error indicates that the compiler cannot guarantee a matching pattern for
one or more possible inputs to a match expression. Guaranteed matches are
required in order to assign values to match expressions, or alternatively,
determine the flow of execution.

Erroneous code example:

enum Terminator {
    HastaLaVistaBaby,
    TalkToMyHand,
}

let x = Terminator::HastaLaVistaBaby;

match x { // error: non-exhaustive patterns: `HastaLaVistaBaby` not covered
    Terminator::TalkToMyHand => {}
}

If you encounter this error you must alter your patterns so that every possible
value of the input type is matched. For types with a small number of variants
(like enums) you should probably cover all cases explicitly. Alternatively, the
underscore `_` wildcard pattern can be added after all other patterns to match
"anything else". Example:

enum Terminator {
    HastaLaVistaBaby,
    TalkToMyHand,
}

let x = Terminator::HastaLaVistaBaby;

match x {
    Terminator::TalkToMyHand => {}
    Terminator::HastaLaVistaBaby => {}
}

// or:

match x {
    Terminator::TalkToMyHand => {}
    _ => {}
}
```

`match` arms have the following structure:

```text
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
```

## `()` Unit Type

In situations where we don't want to perform any action, such as in the `false`
arm, we can use the empty unit type `()`. We'll explore this more as we progress
through the course.

# Summary

In this section, we:

- Explored `for` loop expressions.
- Examined `match` expressions.
- Learned how the Rust compiler provides helpful error messages.
- Were introduced to the empty unit type `()`.

# Exercise

- Replace the `match` expression with an [`if`] expression.

<details>
<summary>Solution</summary>

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
                And he sometimes gets so little that there’s none of him at all.";

    for line in poem.lines() {
        if line.contains(pattern) {
            println!("{line}");
        }
    }
}
```

</details>

# Next

Take a break if you need, and then let's continue!

[`if`]:
  https://doc.rust-lang.org/reference/expressions/if-expr.html#if-expressions
[`if let`]:
  https://doc.rust-lang.org/reference/expressions/if-expr.html#if-let-expressions
[loop expressions]:
  https://doc.rust-lang.org/reference/expressions/loop-expr.html
[`match` expressions]:
  https://doc.rust-lang.org/reference/expressions/match-expr.html
[_scrutinee_]: https://doc.rust-lang.org/reference/glossary.html#scrutinee
