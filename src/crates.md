# Crates

Crates are the smallest units of code that the Rust compiler processes at a
time. For instance, the grep program we're developing is recognized as a crate
by the Rust compiler.

## Types of Crates

Crates are available in two types: binary and library. The compiler identifies
the type of crate being compiled based on the presence of either a `main.rs` or
`lib.rs` file.

### Binary Crates

Binary crates contain a `main.rs` file in the `src` directory, feature a `main`
function, and are compiled into an executable.

### Library Crates

Library crates contain a `lib.rs` file in the `src` directory, lack a main
function, and do not compile into an executable. Instead, they are designed to
be shared with other projects.
