# Types

Let's take a moment to discuss the different types we've encountered:

- The empty unit type `()` is used when a function or expression does not return
  a value. It's essentially a placeholder indicating the absence of a meaningful
  value.
- String slices `&str` are references to a portion of a string. They allow you
  to work with parts of a string without needing to own the entire string,
  making them efficient and flexible.
- Signed 32-bit integers `i32` are integers that can store both positive and
  negative values within a specific range. They are commonly used for numerical
  operations where the size of the number is known and fits within the 32-bit
  limit.

Understanding these types will help us write more efficient and effective Rust
code as we continue to build our rustle program.

## String Slice `str`

The most interesting type is probably `str`. `str` is a primitive string type in
Rust, and it's usually seen in its borrowed[^1] form `&str`.

You can think of a string slice as an object with two components:

| field | value              |
| ----- | ------------------ |
| `ptr` | `address`          |
| `len` | `unsigned integer` |

Imagine a block of memory starting at address `0x10` containing the bytes for
the string literal `"rust"`. This block of memory would store the individual
bytes representing each character in the string. In this case, the memory would
contain the bytes for `'r'`, `'u'`, `'s'`, and `'t'`, sequentially stored
starting from address 0x10.

This visualization helps in understanding how string literals are stored and
accessed in memory:

| 0x10 | 0x11 | 0x12 | 0x13 |
| ---- | ---- | ---- | ---- |
| r    | u    | s    | t    |

When we bind the string literal `"rust"` to the variable `language`:

```rust,noplayground
let language = "rust";
```

The memory would look like:

| field | value  |
| ----- | ------ |
| `ptr` | `0x10` |
| `len` | `4`    |

## Primitive Types

Here are some additional [primitive types] in Rust:

| type    | description                                                                                                            |
| ------- | ---------------------------------------------------------------------------------------------------------------------- |
| array   | A fixed-size array, denoted `[T; N]`, for the element type, `T`, and the non-negative compile-time constant size, `N`. |
| `bool`  | The boolean type.                                                                                                      |
| `char`  | A character type.                                                                                                      |
| `f32`   | A 32-bit floating-point type (specifically, the "binary32" type defined in IEEE 754-2008).                             |
| `f64`   | A 64-bit floating-point type (specifically, the "binary64" type defined in IEEE 754-2008).                             |
| `i8`    | The 8-bit signed integer type.                                                                                         |
| `i16`   | The 16-bit signed integer type.                                                                                        |
| `i32`   | The 32-bit signed integer type.                                                                                        |
| `i64`   | The 64-bit signed integer type.                                                                                        |
| `i128`  | The 128-bit signed integer type.                                                                                       |
| `isize` | The pointer-sized signed integer type.                                                                                 |
| `str`   | String slices.                                                                                                         |
| `u8`    | The 8-bit unsigned integer type.                                                                                       |
| `u16`   | The 16-bit unsigned integer type.                                                                                      |
| `u32`   | The 32-bit unsigned integer type.                                                                                      |
| `u64`   | The 64-bit unsigned integer type.                                                                                      |
| `u128`  | The 128-bit unsigned integer type.                                                                                     |
| `usize` | The pointer-sized unsigned integer type.                                                                               |

## User-defined Types

In addition to the primitive types, Rust supports [user-defined types]. We'll
cover them in more detail as we build our rustle program. But to satisfy your
curiosity, some of the user-defined types include:

| type     | description                                                                                                  |
| -------- | ------------------------------------------------------------------------------------------------------------ |
| `struct` | A heterogeneous product of other types, called the _fields_ of the type.[^2]                                 |
| `enum`   | An enumerated type is a nominal, heterogeneous disjoint union type, denoted by the name of an enum item.[^3] |
| `union`  | A union type is a nominal, heterogeneous C-like union, denoted by the name of a union item.                  |

These user-defined types allow for more complex and expressive code, enabling
you to model real-world concepts more effectively. We'll explore these in
greater depth as we progress through our project.

______________________________________________________________________

[^1]: We'll explore what borrowing means during the course.

[^2]: `struct` types are analogous to `struct` types in C.

[^3]: The `enum` type is analogous to a data constructor declaration in Haskell,
    or a _pick ADT_ in Limbo.

[primitive types]: https://doc.rust-lang.org/std/index.html#primitives
[user-defined types]: https://doc.rust-lang.org/reference/types.html
