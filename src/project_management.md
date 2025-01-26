# Project Management

As your programs grow in size, it becomes crucial to organize your code
effectively. By grouping related functionalities and separating distinct
features, you make it easier to locate the code responsible for a specific
feature and to modify how that feature operates. Let's look at how a Rust
project can be organized.

Paraphrasing the Rust documentation, Rust offers several features to help you
manage your code's organization, including which details are exposed, which are
private, and what names are in each scope of your programs. These features,
often collectively referred to as the module system, include:

- **Packages**: A Cargo feature that lets you build, test, and share crates.
- **Crates**: A tree of modules that produces a library or executable.
- **Modules** and **use**: Allow you to control the organization, scope, and
  privacy of paths.
- **Paths**: A way of naming an item, such as a struct, function, or module.

All the details about managing projects can be found in the [Managing Growing
Projects with Packages, Crates, and Modules] section of [The Rust Programming
Language] documentation.

[Managing Growing Projects with Packages, Crates, and Modules]:
  https://doc.rust-lang.org/book/ch07-00-managing-growing-projects-with-packages-crates-and-modules.html#managing-growing-projects-with-packages-crates-and-modules
[The Rust Programming Language]: https://doc.rust-lang.org/book/title-page.html
