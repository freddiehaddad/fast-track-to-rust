# Traits

In the previous section, we specified trait bounds to restrict the types that
can be used with our `Interval`. Let's formalize what a trait is:

A trait is officially defined as a collection of methods for an unknown type:
`Self`.[^1]

In simpler terms, a trait allows you to define shared behavior in an abstract
way. It specifies a set of methods that a type must implement, similar to
interfaces in other programming languages.

The [Rust By Example] book includes many examples of [traits] and explores their
usage in depth. Reviewing this material is highly recommended.

> The compiler can automatically provide basic implementations for certain
> traits using the derive (`#[derive]`) [attribute]. However, if more complex
> behavior is needed, these traits can still be manually implemented. Here is a
> list of derivable traits:

- Comparison traits: [`Eq`], [`PartialEq`], Ord, [`PartialOrd`].
- [`Clone`], to create `T` from `&T` via a copy.
- [`Copy`], to give a type _copy semantics_ instead of _move semantics_.
- [`Hash`], to compute a hash from `&T`.
- [`Default`], to create an empty instance of a data type.
- [`Debug`], to format a value using the `{:?}` formatter.

## Deriving a Trait

Someone using our `Interval` might find it helpful to have an easy way to
compare them. For instance, a user may want to check if two intervals are equal.
The [`PartialEq`] trait, part of the [`cmp`] module, specifies two methods that
any type must define to implement the `PartialEq` trait.

```rust,noplayground
pub trait PartialEq<Rhs = Self>
where
    Rhs: ?Sized,
{
    // Required method
    fn eq(&self, other: &Rhs) -> bool;

    // Provided method
    fn ne(&self, other: &Rhs) -> bool { ... }
}
```

> Notice the comment above the `ne` method that says "Provided method." This
> indicates that if we manually implement this trait, we only need to provide an
> implementation for `eq`.

Here's our revised `Interval` with support for `PartialEq`, thanks to the derive
attribute and the Rust compiler. Give it a try!

> The `Debug` trait has also been derived, enabling intervals to be printed in a
> programmer-facing context using the `:?` operator. This common pattern in Rust
> provides more helpful output in the program below.

````rust
use interval::Interval;

pub mod interval {
#     /// A list specifying general categories of Interval errors.
    #[derive(Debug)]
    pub enum IntervalError {
#         /// Start is not less than or equal to end
#         StartEndRangeInvalid,
#         /// Two intervals to be merged do not overlap
#         NonOverlappingInterval,
#     }

#     /// A closed-interval [`start`, `end`] type used for representing a range of
#     /// values between `start` and `end` inclusively.
#     ///
#     /// # Examples
#     ///
#     /// You can create an `Interval` using `new`.
#     ///
#     /// ```rust
#     /// let interval = Interval::new(1, 10).unwrap();
#     /// assert_eq!(interval.start, 1);
#     /// assert_eq!(interval.end, 10);
#     /// ```
    #[derive(Debug, PartialEq)]
    pub struct Interval<T> {
#         pub start: T,
#         pub end: T,
#     }
#
#     impl<T: Copy + PartialOrd> Interval<T> {
#         /// Creates a new `Interval` set to `start` and `end`.
#         ///
#         /// # Examples
#         ///
#         /// ```rust
#         /// let interval = Interval::new(1, 10).unwrap();
#         /// assert_eq!(interval.start, 1);
#         /// assert_eq!(interval.end, 10);
#         /// ```
#         pub fn new(start: T, end: T) -> Result<Self, IntervalError> {
#             if start <= end {
#                 Ok(Self { start, end })
#             } else {
#                 Err(IntervalError::StartEndRangeInvalid)
#             }
#         }
#
#         /// Checks if two intervals overlap. Overlapping intervals have at least
#         /// one point in common.
#         ///
#         /// # Examples
#         ///
#         /// ```rust
#         /// let a = Interval::new(1, 3).unwrap();
#         /// let b = Interval::new(3, 5).unwrap();
#         /// assert_eq!(a.overlaps(&b), true);
#         /// assert_eq!(b.overlaps(&a), true);
#         /// ```
#         ///
#         /// ```rust
#         /// let a = Interval::new(1, 5).unwrap();
#         /// let b = Interval::new(2, 4).unwrap();
#         /// assert_eq!(a.overlaps(&b), true);
#         /// assert_eq!(b.overlaps(&a), true);
#         /// ```
#         ///
#         /// ```rust
#         /// let a = Interval::new(1, 3).unwrap();
#         /// let b = Interval::new(4, 6).unwrap();
#         /// assert_eq!(a.overlaps(&b), false);
#         /// assert_eq!(b.overlaps(&a), true);
#         /// ```
#         pub fn overlaps(&self, other: &Interval<T>) -> bool {
#             self.end >= other.start
#         }
#
#         /// Merges two intervals returning a new `Interval`.
#         ///
#         /// The merged `Interval` range includes the union of ranges from each
#         /// `Interval`.
#         ///
#         /// # Examples
#         ///
#         /// ```rust
#         /// let a = Interval::new(1, 3).unwrap();
#         /// let b = Interval::new(3, 5).unwrap();
#         /// let c = a.merge(&b).unwrap();
#         /// assert_eq!(c.start, 1);
#         /// assert_eq!(c.end, 5);
#         /// ```
#         pub fn merge(&self, other: &Self) -> Result<Self, IntervalError> {
#             if self.overlaps(other) {
#                 Ok(Self {
#                     start: self.start,
#                     end: other.end,
#                 })
#             } else {
#                 Err(IntervalError::NonOverlappingInterval)
#             }
#         }
#     }
}

fn main() {
    let a = Interval::new(1, 5).unwrap(); // unwrap returns the Ok value or panics
    let b = Interval::new(1, 5).unwrap();
    let c = Interval::new(2, 4).unwrap();

    // Comparing intervals with eq and ne.
    println!("{:?} == {:?} => {}", a, b, a.eq(&b));
    println!("{:?} == {:?} => {}", a, c, a.eq(&c));
    println!("{:?} != {:?} => {}", a, b, a.ne(&b));
    println!("{:?} != {:?} => {}", a, c, a.ne(&c));

    // Rust supports operator overloading too!
    println!("{:?} == {:?} => {}", a, b, a == b);
    println!("{:?} != {:?} => {}", a, c, a != c);
}
````

> The [`unwrap()`] method is implemented by `Result` and other types like
> `Option`. For `Result`, it returns the `Ok` value or panics if the value is
> `Err`. Its usage is generally discouraged and is only used here for brevity.

> Operator Overloading
>
> You might have noticed the comparison between intervals using `==` and `!=`.
> This works because Rust includes support for operator overloading! You can
> find all the detailed information in the [Operator Overloading] section of the
> [Rust by Example] book and in the [`ops`] module.

## Implementing a Trait

Just as comparing traits for equality can be helpful, so can generating a
user-facing representation of an `Interval`. By implementing the [`Display`]
trait, we can achieve this quite efficiently. Additionally, implementing the
`Display` trait automatically implements the [`ToString`] trait, enabling the
use of the `to_string()` method. Let's enhance our `Interval` code to print an
interval in the form `[x, y]`.

Here's the trait definition for `Display`:

```rust,noplayground
pub trait Display {
    // Required method
    fn fmt(&self, f: &mut Formatter<'_>) -> Result<(), Error>;
}
```

> The `<'_>` suffix next to `Formatter` is known as _lifetime annotation
> syntax_.[^2]

Here's our revised `Interval` with the `Display` trait implemented. Give it a
try!

````rust
use interval::Interval;

pub mod interval {
#     /// A list specifying general categories of Interval errors.
#     #[derive(Debug)]
#     pub enum IntervalError {
#         /// Start is not less than or equal to end
#         StartEndRangeInvalid,
#         /// Two intervals to be merged do not overlap
#         NonOverlappingInterval,
#     }
#
#     /// A closed-interval [`start`, `end`] type used for representing a range of
#     /// values between `start` and `end` inclusively.
#     ///
#     /// # Examples
#     ///
#     /// You can create an `Interval` using `new`.
#     ///
#     /// ```rust
#     /// let interval = Interval::new(1, 10).unwrap();
#     /// assert_eq!(interval.start, 1);
#     /// assert_eq!(interval.end, 10);
#     /// ```
#     #[derive(Debug, PartialEq)]
#     pub struct Interval<T> {
#         pub start: T,
#         pub end: T,
#     }
#
    use std::fmt;
    impl<T: fmt::Display> fmt::Display for Interval<T> {
        fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> fmt::Result {
            write!(f, "[{}, {}]", self.start, self.end)
        }
    }
#
#     impl<T: Copy + PartialOrd> Interval<T> {
#         /// Creates a new `Interval` set to `start` and `end`.
#         ///
#         /// # Examples
#         ///
#         /// ```rust
#         /// let interval = Interval::new(1, 10).unwrap();
#         /// assert_eq!(interval.start, 1);
#         /// assert_eq!(interval.end, 10);
#         /// ```
#         pub fn new(start: T, end: T) -> Result<Self, IntervalError> {
#             if start <= end {
#                 Ok(Self { start, end })
#             } else {
#                 Err(IntervalError::StartEndRangeInvalid)
#             }
#         }
#
#         /// Checks if two intervals overlap. Overlapping intervals have at least
#         /// one point in common.
#         ///
#         /// # Examples
#         ///
#         /// ```rust
#         /// let a = Interval::new(1, 3).unwrap();
#         /// let b = Interval::new(3, 5).unwrap();
#         /// assert_eq!(a.overlaps(&b), true);
#         /// assert_eq!(b.overlaps(&a), true);
#         /// ```
#         ///
#         /// ```rust
#         /// let a = Interval::new(1, 5).unwrap();
#         /// let b = Interval::new(2, 4).unwrap();
#         /// assert_eq!(a.overlaps(&b), true);
#         /// assert_eq!(b.overlaps(&a), true);
#         /// ```
#         ///
#         /// ```rust
#         /// let a = Interval::new(1, 3).unwrap();
#         /// let b = Interval::new(4, 6).unwrap();
#         /// assert_eq!(a.overlaps(&b), false);
#         /// assert_eq!(b.overlaps(&a), true);
#         /// ```
#         pub fn overlaps(&self, other: &Interval<T>) -> bool {
#             self.end >= other.start
#         }
#
#         /// Merges two intervals returning a new `Interval`.
#         ///
#         /// The merged `Interval` range includes the union of ranges from each
#         /// `Interval`.
#         ///
#         /// # Examples
#         ///
#         /// ```rust
#         /// let a = Interval::new(1, 3).unwrap();
#         /// let b = Interval::new(3, 5).unwrap();
#         /// let c = a.merge(&b).unwrap();
#         /// assert_eq!(c.start, 1);
#         /// assert_eq!(c.end, 5);
#         /// ```
#         pub fn merge(&self, other: &Self) -> Result<Self, IntervalError> {
#             if self.overlaps(other) {
#                 Ok(Self {
#                     start: self.start,
#                     end: other.end,
#                 })
#             } else {
#                 Err(IntervalError::NonOverlappingInterval)
#             }
#         }
#     }
}

fn main() {
    let a = Interval::new(1, 5).unwrap(); // unwrap returns the Ok value or panics
    let b = Interval::new(1, 5).unwrap();
    let b_str = b.to_string();
    let c = Interval::new(2, 4).unwrap();

    println!(
        "Comparisons between three intervals: {}, {}, {}...",
        a, b_str, c
    );

    // Comparing intervals with eq and ne.
    println!("{} == {} => {}", a, b, a.eq(&b));
    println!("{} == {} => {}", a, c, a.eq(&c));
    println!("{} != {} => {}", a, b, a.ne(&b));
    println!("{} != {} => {}", a, c, a.ne(&c));

    // Rust supports operator overloading too!
    println!("{} == {} => {}", a, b, a == b);
    println!("{} != {} => {}", a, c, a != c);
}
````

# Summary

- A trait defines shared behavior abstractly, specifying methods a type must
  implement, similar to interfaces in other languages.
- The compiler can automatically provide basic implementations for certain
  traits using the `#[derive]` attribute.
- Rust includes support for operator overloading.

# Exercise

- Implement the [`PartialOrd`] trait for `Interval`. To keep the code simple,
  you can return `None` for overlapping intervals.

````rust,editable
use interval::Interval;

pub mod interval {
    /// A list specifying general categories of Interval errors.
    #[derive(Debug)]
    pub enum IntervalError {
        /// Start is not less than or equal to end
        StartEndRangeInvalid,
        /// Two intervals to be merged do not overlap
        NonOverlappingInterval,
    }

    /// A closed-interval [`start`, `end`] type used for representing a range of
    /// values between `start` and `end` inclusively.
    ///
    /// # Examples
    ///
    /// You can create an `Interval` using `new`.
    ///
    /// ```rust
    /// let interval = Interval::new(1, 10).unwrap();
    /// assert_eq!(interval.start, 1);
    /// assert_eq!(interval.end, 10);
    /// ```
    #[derive(Debug, PartialEq)]
    pub struct Interval<T> {
        pub start: T,
        pub end: T,
    }

    use std::fmt;
    impl<T: fmt::Display> fmt::Display for Interval<T> {
        fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> fmt::Result {
            write!(f, "[{}, {}]", self.start, self.end)
        }
    }

    impl<T: Copy + PartialOrd> Interval<T> {
        /// Creates a new `Interval` set to `start` and `end`.
        ///
        /// # Examples
        ///
        /// ```rust
        /// let interval = Interval::new(1, 10).unwrap();
        /// assert_eq!(interval.start, 1);
        /// assert_eq!(interval.end, 10);
        /// ```
        pub fn new(start: T, end: T) -> Result<Self, IntervalError> {
            if start <= end {
                Ok(Self { start, end })
            } else {
                Err(IntervalError::StartEndRangeInvalid)
            }
        }

        /// Checks if two intervals overlap. Overlapping intervals have at least
        /// one point in common.
        ///
        /// # Examples
        ///
        /// ```rust
        /// let a = Interval::new(1, 3).unwrap();
        /// let b = Interval::new(3, 5).unwrap();
        /// assert_eq!(a.overlaps(&b), true);
        /// assert_eq!(b.overlaps(&a), true);
        /// ```
        ///
        /// ```rust
        /// let a = Interval::new(1, 5).unwrap();
        /// let b = Interval::new(2, 4).unwrap();
        /// assert_eq!(a.overlaps(&b), true);
        /// assert_eq!(b.overlaps(&a), true);
        /// ```
        ///
        /// ```rust
        /// let a = Interval::new(1, 3).unwrap();
        /// let b = Interval::new(4, 6).unwrap();
        /// assert_eq!(a.overlaps(&b), false);
        /// assert_eq!(b.overlaps(&a), true);
        /// ```
        pub fn overlaps(&self, other: &Interval<T>) -> bool {
            self.end >= other.start
        }

        /// Merges two intervals returning a new `Interval`.
        ///
        /// The merged `Interval` range includes the union of ranges from each
        /// `Interval`.
        ///
        /// # Examples
        ///
        /// ```rust
        /// let a = Interval::new(1, 3).unwrap();
        /// let b = Interval::new(3, 5).unwrap();
        /// let c = a.merge(&b).unwrap();
        /// assert_eq!(c.start, 1);
        /// assert_eq!(c.end, 5);
        /// ```
        pub fn merge(&self, other: &Self) -> Result<Self, IntervalError> {
            if self.overlaps(other) {
                Ok(Self {
                    start: self.start,
                    end: other.end,
                })
            } else {
                Err(IntervalError::NonOverlappingInterval)
            }
        }
    }
}

fn main() {
    use std::cmp::Ordering::{Equal, Greater, Less};

    let a = Interval::new(0, 1).unwrap();
    let b = Interval::new(0, 1).unwrap();
    assert_eq!(a.partial_cmp(&b), Some(Equal));
    assert_eq!(b.partial_cmp(&a), Some(Equal));

    let a = Interval::new(0, 1).unwrap();
    let b = Interval::new(2, 3).unwrap();
    assert_eq!(a.partial_cmp(&b), Some(Less));
    assert_eq!(b.partial_cmp(&a), Some(Greater));

    println!("Nice Work!");
}
````

<details>

<summary>Solution</summary>

````rust
# use interval::Interval;
#
pub mod interval {
#     /// A list specifying general categories of Interval errors.
#     #[derive(Debug)]
#     pub enum IntervalError {
#         /// Start is not less than or equal to end
#         StartEndRangeInvalid,
#         /// Two intervals to be merged do not overlap
#         NonOverlappingInterval,
#     }
#
#     /// A closed-interval [`start`, `end`] type used for representing a range of
#     /// values between `start` and `end` inclusively.
#     ///
#     /// # Examples
#     ///
#     /// You can create an `Interval` using `new`.
#     ///
#     /// ```rust
#     /// let interval = Interval::new(1, 10).unwrap();
#     /// assert_eq!(interval.start, 1);
#     /// assert_eq!(interval.end, 10);
#     /// ```
#     #[derive(Debug, PartialEq)]
#     pub struct Interval<T> {
#         pub start: T,
#         pub end: T,
#     }
#
#     use std::fmt;
#     impl<T: fmt::Display> fmt::Display for Interval<T> {
#         fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> fmt::Result {
#             write!(f, "[{}, {}]", self.start, self.end)
#         }
#     }
#
    use std::cmp::Ordering;
    impl<T: PartialEq + PartialOrd> PartialOrd for Interval<T> {
        fn partial_cmp(&self, other: &Self) -> Option<Ordering> {
            if self == other {
                Some(Ordering::Equal)
            } else if self.end < other.start {
                Some(Ordering::Less)
            } else if self.start > other.end {
                Some(Ordering::Greater)
            } else {
                None // Intervals overlap
            }
        }
    }
#
#     impl<T: Copy + PartialOrd> Interval<T> {
#         /// Creates a new `Interval` set to `start` and `end`.
#         ///
#         /// # Examples
#         ///
#         /// ```rust
#         /// let interval = Interval::new(1, 10).unwrap();
#         /// assert_eq!(interval.start, 1);
#         /// assert_eq!(interval.end, 10);
#         /// ```
#         pub fn new(start: T, end: T) -> Result<Self, IntervalError> {
#             if start <= end {
#                 Ok(Self { start, end })
#             } else {
#                 Err(IntervalError::StartEndRangeInvalid)
#             }
#         }
#
#         /// Checks if two intervals overlap. Overlapping intervals have at least
#         /// one point in common.
#         ///
#         /// # Examples
#         ///
#         /// ```rust
#         /// let a = Interval::new(1, 3).unwrap();
#         /// let b = Interval::new(3, 5).unwrap();
#         /// assert_eq!(a.overlaps(&b), true);
#         /// assert_eq!(b.overlaps(&a), true);
#         /// ```
#         ///
#         /// ```rust
#         /// let a = Interval::new(1, 5).unwrap();
#         /// let b = Interval::new(2, 4).unwrap();
#         /// assert_eq!(a.overlaps(&b), true);
#         /// assert_eq!(b.overlaps(&a), true);
#         /// ```
#         ///
#         /// ```rust
#         /// let a = Interval::new(1, 3).unwrap();
#         /// let b = Interval::new(4, 6).unwrap();
#         /// assert_eq!(a.overlaps(&b), false);
#         /// assert_eq!(b.overlaps(&a), true);
#         /// ```
#         pub fn overlaps(&self, other: &Interval<T>) -> bool {
#             self.end >= other.start
#         }
#
#         /// Merges two intervals returning a new `Interval`.
#         ///
#         /// The merged `Interval` range includes the union of ranges from each
#         /// `Interval`.
#         ///
#         /// # Examples
#         ///
#         /// ```rust
#         /// let a = Interval::new(1, 3).unwrap();
#         /// let b = Interval::new(3, 5).unwrap();
#         /// let c = a.merge(&b).unwrap();
#         /// assert_eq!(c.start, 1);
#         /// assert_eq!(c.end, 5);
#         /// ```
#         pub fn merge(&self, other: &Self) -> Result<Self, IntervalError> {
#             if self.overlaps(other) {
#                 Ok(Self {
#                     start: self.start,
#                     end: other.end,
#                 })
#             } else {
#                 Err(IntervalError::NonOverlappingInterval)
#             }
#         }
#     }
}
#
# fn main() {
#     use std::cmp::Ordering::{Equal, Greater, Less};
#
#     let a = Interval::new(0, 1).unwrap();
#     let b = Interval::new(0, 1).unwrap();
#     assert_eq!(a.partial_cmp(&b), Some(Equal));
#     assert_eq!(b.partial_cmp(&a), Some(Equal));
#
#     let a = Interval::new(0, 1).unwrap();
#     let b = Interval::new(2, 3).unwrap();
#     assert_eq!(a.partial_cmp(&b), Some(Less));
#     assert_eq!(b.partial_cmp(&a), Some(Greater));
#
#     println!("Nice Work!");
# }
````

</details>

# Next

Let's combine all the code and take a brief moment to explain the attributes.

[Rust By Example]: https://doc.rust-lang.org/rust-by-example/
[traits]: https://doc.rust-lang.org/rust-by-example/trait.html
[derive]: https://doc.rust-lang.org/reference/attributes/derive.html
[attribute]: https://doc.rust-lang.org/reference/attributes.html
[`PartialOrd`]: https://doc.rust-lang.org/std/cmp/trait.PartialOrd.html
[`PartialEq`]: https://doc.rust-lang.org/std/cmp/trait.PartialEq.html
[`Eq`]: https://doc.rust-lang.org/std/cmp/trait.Eq.html
[`Clone`]: https://doc.rust-lang.org/std/clone/trait.Clone.html
[`Copy`]: https://doc.rust-lang.org/std/marker/trait.Copy.html
[`Hash`]: https://doc.rust-lang.org/std/hash/trait.Hash.html
[`Default`]: https://doc.rust-lang.org/std/default/trait.Default.html
[`Debug`]: https://doc.rust-lang.org/std/fmt/trait.Debug.html
[`cmp`]: https://doc.rust-lang.org/std/cmp/index.html
[`unwrap()`]:
  https://doc.rust-lang.org/std/result/enum.Result.html#method.unwrap
[Operator Overloading]: https://doc.rust-lang.org/rust-by-example/trait/ops.html
[`ops`]: https://doc.rust-lang.org/std/ops/index.html
[`Display`]: https://doc.rust-lang.org/std/fmt/trait.Display.html
[`ToString`]: https://doc.rust-lang.org/std/string/trait.ToString.html
[`.to_string()`]:
  https://doc.rust-lang.org/std/string/trait.ToString.html#tymethod.to_string
[Validating References with Lifetimes]:
  https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html
[Rust Programming Language]: https://doc.rust-lang.org/stable/book/
[Lifetimes]: https://doc.rust-lang.org/rust-by-example/scope/lifetime.html

---

[^1]: https://doc.rust-lang.org/rust-by-example/trait.html

[^2]:
    This is a separate topic that you can learn more about in the [Validating
    References with Lifetimes] section of the [Rust Programming Language] and
    the [Lifetimes] section of [Rust By Example].
