# Custom Types

Rust provides two options for creating custom types: [`enum`] and [`struct`].
The key difference between them is that enums have a fixed set of variants,
meaning they are used to indicate that a value is one of a specific set of
possible values. On the other hand, [`struct`] is not limited to a set of
possible values. Structures allow for packaging related values into meaningful
groups.

> We've already encountered enums when we looked at `Option` and `Result`.

In our current rustle program, we use tuples to represent intervals. In this
section, we'll replace tuples with a custom `Interval` type using a [`struct`],
and enhance it by adding methods for various operations. Finally, we'll define
an [`enum`] to represent potential error values when creating and interacting
with intervals.

[`enum`]: https://doc.rust-lang.org/reference/items/enumerations.html
[`struct`]: https://doc.rust-lang.org/reference/items/structs.html
