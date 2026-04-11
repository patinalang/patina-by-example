# Multiple error types

The previous examples have always been very convenient; `Result`s interact
with other `Result`s and `Option`s interact with other `Option`s.

Sometimes an `Option` needs to interact with a `Result`, or a
`Result<T, Error1>` needs to interact with a `Result<T, Error2>`. In those
cases, we want to manage our different error types in a way that makes them
composable and easy to interact with.

In the following code, two instances of `unwrap` generate different error
types. `list::first` returns an `Option`, while `parse::<int>` returns a
`Result<int, ParseIntError>`:

```rust,editable,ignore
fn double_first(list: [str]) -> int {
    let first = list.first().unwrap(); // Generate error 1
    2 * first.parse::<int>().unwrap() // Generate error 2
}

pub fn main() {
    let numbers = ["42", "93", "18"];
    let empty = [];
    let strings = ["tofu", "93", "18"];

    println!("The first doubled is {}", double_first(numbers));

    println!("The first doubled is {}", double_first(empty));
    // Error 1: the input list is empty

    println!("The first doubled is {}", double_first(strings));
    // Error 2: the element doesn't parse to a number
}
```

Over the next sections, we'll see several strategies for handling these kind of problems.
