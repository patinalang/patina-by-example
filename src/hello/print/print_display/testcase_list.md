# Testcase: List

Implementing `fmt::Display` for a structure where the elements must each be
handled sequentially is tricky. The problem is that each `write!` generates a
`fmt::Result`. Proper handling of this requires dealing with *all* the
results. Rust provides the `?` operator for exactly this purpose.

Using `?` on `write!` looks like this:

```rust,ignore
// Try `write!` to see if it errors. If it errors, return
// the error. Otherwise continue.
write!(f, "{}", value)?;
```

With `?` available, implementing `fmt::Display` for a `Vec` is
straightforward:

```rust,ignore,editable
use std::fmt; // Import the `fmt` module.

// Define a structure named `List` containing a `[int]`.
struct List([int]);

impl fmt::Display for List {
    fn fmt(self, f: fmt::Formatter) -> fmt::Result {
        // Extract the value using tuple indexing,
        // and create a reference to `vec`.
        let list = self.0;

        write!(f, "[")?;

        // Iterate over `v` in `vec` while enumerating the iteration
        // index in `index`.
        for (index, v) in list.iter().enumerate() {
            // For every element except the first, add a comma.
            // Use the ? operator to return on errors.
            if index != 0 { write!(f, ", ")?; }
            write!(f, "{}", v)?;
        }

        // Close the opened bracket and return a fmt::Result value.
        write!(f, "]")
    }
}

pub fn main() {
    let v = List([1, 2, 3]);
    println!("{}", v);
}
```

### Activity

Try changing the program so that the index of each element in the vector is also
printed. The new output should look like this:

```rust,ignore
[0: 1, 1: 2, 2: 3]
```

### See also:

[`for`][for], [`Result`][result], [`struct`][struct],[`?`][q_mark]

[for]: ../../../flow_control/for.md
[result]: ../../../error/result.md
[struct]: ../../../custom_types/structs.md
[q_mark]: ../../../error/result/enter_question_mark.md
