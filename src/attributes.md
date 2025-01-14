# Attributes

Throughout this course, we've been using [attributes], but we haven't delved
into them yet. Let's briefly discuss the attributes you've encountered and
provide some useful resources for further exploration.

At a high level, an attribute is metadata applied to a crate, module, or item.
This metadata serves various purposes. Attributes follow this syntax:

```console
InnerAttribute :
   # ! [ Attr ]

OuterAttribute :
   # [ Attr ]

Attr :
      SimplePath AttrInput?
   | unsafe ( SimplePath AttrInput? )

AttrInput :
      DelimTokenTree
   | = Expression
```

Attributes come in two forms: `InnerAttribute` and `OuterAttribute`. The
difference between them is that an `InnerAttribute` includes a bang (`!`)
between the `#` and the `[`. An `InnerAttribute` applies to the item it is
declared within, while an `OuterAttribute` applies to whatever follows it.

## Inner Attributes

For example, in our grep program, we've used the inner attribute
`#![allow(unused_imports)]`. By default, the Rust compiler issues warnings if
your program imports packages via `use` but doesn't actually utilize them. Our
program imports the `File` module with `use std::fs::File;`, but since we are
using `Cursor` for development and have commented out the file I/O code, the
compiler complains about the unused import. This attribute disables that warning
for the crate.

## Outer Attributes

Recall in the last section, we derived the `Debug` and `PartialEq` traits. We
achieved this using the `#[derive(...)]` attribute. Notice how this attribute is
placed directly before the definition of the `Interval` `struct`. This is an
example of an `OuterAttribute`.

```rust,noplayground
#[derive(Debug, PartialEq)]
pub struct Interval<T> {
    pub start: T,
    pub end: T,
}
```

## Classification of Attributes

Attributes are commonly grouped into the following categories:

- [Built-in attributes]
- [Macro attributes]
- [Derive macro helper attributes]
- [Tool attributes]

# Summary

[The Rust Reference] provides an in-depth look at [attributes], and you are
encouraged to explore it for a comprehensive understanding of their uses.
Additionally, [Rust By Example] offers extra material and illustrates various
[use cases].

[attributes]: https://doc.rust-lang.org/reference/attributes.html
[Built-in attributes]:
  https://doc.rust-lang.org/reference/attributes.html#built-in-attributes-index
[Macro attributes]:
  https://doc.rust-lang.org/reference/procedural-macros.html#attribute-macros
[Derive macro helper attributes]:
  https://doc.rust-lang.org/reference/procedural-macros.html#derive-macro-helper-attributes
[Tool attributes]:
  https://doc.rust-lang.org/reference/attributes.html#tool-attributes
[The Rust Reference]: https://doc.rust-lang.org/reference/
[Rust By Example]: https://doc.rust-lang.org/rust-by-example/
[use cases]: https://doc.rust-lang.org/rust-by-example/attribute.html

# Next

Attributes offer even greater utility, which we'll explore through command line
argument parsing!
