# Modules

_Modules_ help us organize code within a crate for better readability and easy
reuse. They also allow us to control the _privacy_ of items, as code within a
module is private by default. Private items are internal implementation details
not accessible from outside. We can choose to make modules and their items
public, exposing them for external code to use and depend on.

Refer to the section on [Defining Modules to Control Scope and Privacy] in the
Rust documentation.

[Defining Modules to Control Scope and Privacy]:
  https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html#defining-modules-to-control-scope-and-privacy

# Summary

In this section, we were introduced to:

- Crates: A tree of modules that produces a library or executable
- Packages: A Cargo feature that lets you build, test, and share crates
- Modules: Let you control the organization of your project

# Next

With a basic understanding of crates, packages, and modules, let's utilize one
for regular expression support.
