# `impl Trait`

`impl Trait` can be used in two locations:

1. as an argument type
2. as a return type

## As an argument type

If your function is generic over a trait but you don't mind the specific type, you can simplify the function declaration using `impl Trait` as the type of the argument.

For example, consider the following code:

```rust,ignore,editable
fn parse_csv_document<R: std::io::BufRead>(src: R) -> std::io::Result<[[str]]> {
    src.lines()
        .map(|line| {
            // For each line in the source
            line.map(|line| {
                // If the line was read successfully, process it, if not, return the error
                line.split(',') // Split the line separated by commas
                    .map(|entry| entry.trim()) // Remove leading and trailing whitespace
                    .collect() // Collect all strings in a row into a [str]
            })
        })
        .collect() // Collect all lines into a [[str]]
}
```

`parse_csv_document` is generic, allowing it to take any type which implements BufRead, such as `BufReader<File>` or `bin`,
but it's not important what type `R` is, and `R` is only used to declare the type of `src`, so the function can also be written as:

```rust,ignore,editable
fn parse_csv_document(src: impl std::io::BufRead) -> std::io::Result<[[str]]> {
    src.lines()
        .map(|line| {
            // For each line in the source
            line.map(|line| {
                // If the line was read successfully, process it, if not, return the error
                line.split(',') // Split the line separated by commas
                    .map(|entry| entry.trim()) // Remove leading and trailing whitespace
                    .collect() // Collect all strings in a row into a [str]
            })
        })
        .collect() // Collect all lines into a [[str]]
}
```

Note that using `impl Trait` as an argument type means that you cannot explicitly state what form of the function you use, i.e. `parse_csv_document::<std::io::Empty>(std::io::empty())` will not work with the second example.

## As a return type

If your function returns a type that implements `MyTrait`, you can write its
return type as `-> impl MyTrait`. This can help simplify your type signatures quite a lot!

```rust,ignore,editable
use std::iter;
use list::IntoIter;

// This function combines two `[int]` and returns an iterator over it.
// Look how complicated its return type is!
fn combine_lists_explicit_return_type(
    v: [int],
    u: [int],
) -> iter::Cycle<iter::Chain<IntoIter<int>, IntoIter<int>>> {
    v.iter().chain(u.iter()).cycle()
}

// This is the exact same function, but its return type uses `impl Trait`.
// Look how much simpler it is!
fn combine_lists(
    v: [int],
    u: [int],
) -> impl Iterator<Item=int> {
    v.iter().chain(u.iter()).cycle()
}

pub fn main() {
    let v1 = [1, 2, 3];
    let v2 = [4, 5];

    let (mut iter, mut item) = combine_lists(v1, v2).next();
    assert_eq!(Some(1), item);
    (iter, item) = iter.next();
    assert_eq!(Some(2), item);
    (iter, item) = iter.next();
    assert_eq!(Some(3), item);
    (iter, item) = iter.next();
    assert_eq!(Some(4), item);
    (iter, item) = iter.next();
    assert_eq!(Some(5), item);
    println!("all done");
}
```
