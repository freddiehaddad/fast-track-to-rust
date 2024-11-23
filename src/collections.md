# Collections

Rust's standard library features some highly powerful and useful data
structures, referred to as [collections]. While we'll explore one of them in
this course, it's definitely worth spending time reviewing the entire
collections library.

The collections are commonly grouped into four categories:

- Sequences: [Vec], [VecDeque], [LinkedList]
- Maps: [HashMap], [BTreeMap]
- Sets: [HashSet], [BTreeSet]
- Misc: [BinaryHeap]

## Grep

Currently, our grep program only outputs the line number and the matching line
for each pattern match. It doesn't yet support printing lines before or after
the match. So far, we haven't had a straightforward way to implement this
functionality, and there are several challenges we need to address.

### Handling Context

The `--before-context` option requires us to print preceding lines when we
encounter a match. This necessitates a mechanism to reference those lines.
However, we should only print each line once to avoid duplicating lines when
consecutive lines within the context contain matches.

Consider the poem from our code:

```text
I have a little shadow that goes in and out with me,
And what can be the use of him is more than I can see.
He is very, very like me from the heels up to the head;
And I see him jump before me, when I jump into my bed.

The funniest thing about him is the way he likes to grow -
Not at all like proper children, which is always very slow;
For he sometimes shoots up taller like an india-rubber ball,
And he sometimes gets so little that there’s none of him at all.
```

If the pattern is "him" and the number of lines to print before each match is
set to 2 (`--before-context=2`), without a method to track printed lines, we
would end up with the following output:

```text
1: I have a little shadow that goes in and out with me,
2: And what can be the use of him is more than I can see.
2: And what can be the use of him is more than I can see.
3: He is very, very like me from the heels up to the head;
4: And I see him jump before me, when I jump into my bed.
5:
4: And I see him jump before me, when I jump into my bed.
5:
6: The funniest thing about him is the way he likes to grow -
7: Not at all like proper children, which is always very slow;
8: For he sometimes shoots up taller like an india-rubber ball,
9: And he sometimes gets so little that there’s none of him at all.
```

This is not the behavior we want from our grep program. Instead, we aim for the
following output:

```text
1: I have a little shadow that goes in and out with me,
2: And what can be the use of him is more than I can see.
3: He is very, very like me from the heels up to the head;
4: And I see him jump before me, when I jump into my bed.
5:
6: The funniest thing about him is the way he likes to grow -
7: Not at all like proper children, which is always very slow;
8: For he sometimes shoots up taller like an india-rubber ball,
9: And he sometimes gets so little that there’s none of him at all.
```

A similar issue arises with `--after-context`. Our solution involves using a
vector and tuples (since we are now familiar with them) to create intervals over
the ranges of lines we want to print.

### Accessing Lines from the Input

The context problem indicates that we need a method to reference a range of
lines around each match. We'll address this by storing the lines in a collection
from the Rust standard library called a vector `Vec<T>`.

### Tracking the Lines Surrounding Each Match

Since we are familiar with them, we can use a `tuple` to represent the starting
and ending intervals around each match. We'll need to track these intervals and
merge overlapping ones to avoid printing the same line multiple times. Once
again, a vector will be useful!

# Next

Let's start coding!

[collections]: https://doc.rust-lang.org/std/collections/index.html
[Vec]: https://doc.rust-lang.org/std/vec/struct.Vec.html
[VecDeque]: https://doc.rust-lang.org/std/collections/struct.VecDeque.html
[LinkedList]: https://doc.rust-lang.org/std/collections/struct.LinkedList.html
[HashMap]:
  https://doc.rust-lang.org/std/collections/hash_map/struct.HashMap.html
[BTreeMap]: https://doc.rust-lang.org/std/collections/struct.BTreeMap.html
[HashSet]:
  https://doc.rust-lang.org/std/collections/hash_set/struct.HashSet.html
[BTreeSet]: https://doc.rust-lang.org/std/collections/struct.BTreeSet.html
[BinaryHeap]: https://doc.rust-lang.org/std/collections/struct.BinaryHeap.html
