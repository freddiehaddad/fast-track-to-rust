# Ownership

Ownership and borrowing are fundamental principles in Rust that ensure memory
safety without needing a garbage collector. Ownership dictates how memory is
managed, while borrowing allows you to reference data without taking ownership.
Understanding these concepts is crucial for writing efficient and safe Rust
programs.

The Rust documentation provides an in-depth exploration of these topics in
[Understanding Ownership], and it's highly recommended to spend some time
reading that material. For now, let's focus on the core principles of ownership.

## Core Principles

Keep these rules about ownership in mind as we progress through the course:

- Each value in Rust has an owner.
- There can only be one owner at a time.
- When the owner goes out of scope, the value is _dropped_.[^1]

[Understanding Ownership]:
  https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html

---

[^1]:
    The term _dropped_ means the memory is freed and the object's lifetime has
    ended.
