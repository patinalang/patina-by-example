# lists

Like tuples, lists can be destructured this way:

```rust,ignore,editable
pub fn main() {
    // Try changing the values in the list!
    let list = [1, -2, 6];

    match list {
        // Binds the second and the third elements to the respective variables
        [0, second, third] =>
            println!("list[0] = 0, list[1] = {}, list[2] = {}", second, third),

        // Single values can be ignored with _
        [1, _, third] => println!(
            "list[0] = 1, list[2] = {} and list[1] was ignored",
            third
        ),

        // You can also bind some and ignore the rest
        [-1, second, ..] => println!(
            "list[0] = -1, list[1] = {} and all the other ones were ignored",
            second
        ),
        // The code below would not compile
        // [-1, second] => ...

        // Or store them in another list (the type depends on
        // that of the value that is being matched against)
        [3, second, tail @ ..] => println!(
            "list[0] = 3, list[1] = {} and the other elements were {:?}",
            second, tail
        ),

        // We have to include a catch-all for match exhaustiveness.
        _ => println!("no list pattern matched"),
    }
}
```

### See also:

[Lists](../../../primitives/lists.md) and [Binding](../binding.md) for `@` sigil
