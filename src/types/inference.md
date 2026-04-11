# Inference

The type inference engine is pretty smart. It does more than looking at the
type of the value expression
during an initialization. It also looks at how the variable is used afterwards
to infer its type. Here's an advanced example of type inference:

```rust,ignore,editable
pub fn main() {
    // The compiler knows that `elem` has type `int`.
    let elem = 1;

    // Create an empty list.
    let mut lst = [];
    // At this point the compiler doesn't know the exact type of `lst`, it
    // just knows that it's a list of something (`[_]`).

    // Prepend `elem` to the list.
    lst = [elem, ..lst]
    // Aha! Now the compiler knows that `lst` is a list of `int`s (`[int]`)
    println!("{:?}", lst);

    // Error! `str` does not fit in `[int]`
    lst = ["hi", ..lst]
}
```

No type annotation of variables was needed, the compiler is happy and so is the
programmer!
