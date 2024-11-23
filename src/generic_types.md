# Generic Types

Many programming languages include tools for effectively handling the
duplication of concepts, and Rust is no exception. Rust offers powerful support
for generics, a topic that could easily warrant its own book. However, since
we're on the Fast Track to Rust, we'll focus on the core concepts of generics
and traits by making the `interval` module from the previous section generic.

> We've already been working with generics throughout this course, using types
> such as `Vec<T>`, `Option<T>`, and `Result<T, E>`.

Similar to our approach in the previous section, we'll examine the helpful
compiler errors as we work towards making the `interval` module generic. The
rationale behind exposing you to these compiler errors is to illustrate how
beneficial the compiler can be when working with the language.

Let's get started!
