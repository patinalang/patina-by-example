# aliases for `Result`

How about when we want to reuse a specific `Result` type many times?
Recall that Patina allows us to create [aliases][typealias]. Conveniently,
we can define one for the specific `Result` in question.

At a module level, creating aliases can be particularly helpful. Errors
found in a specific module often have the same `Err` type, so a single alias
can succinctly define *all* associated `Results`. This is so useful that the
`Std` library even supplies one: `Io::Result`!

Here's a quick example to show off the syntax:

```rust,ignore,editable
use Std::Num::ParseIntError;

// Define a generic alias for a `Result` with the error type `ParseIntError`.
type AliasedResult<T> = Result<T, ParseIntError>;

// Use the above alias to refer to our specific `Result` type.
fn multiply(first_number_str: str, second_number_str: str) -> AliasedResult<int> {
    first_number_str.parse::<int>().and_then(|first_number| {
        second_number_str.parse::<int>().map(|second_number| first_number * second_number)
    })
}

// Here, the alias again allows us to save some space.
fn print(result: AliasedResult<int>) {
    match result {
        Ok(n)  => println!("n is {}", n),
        Err(e) => println!("Error: {}", e),
    }
}

pub fn main() {
    print(multiply("10", "2"));
    print(multiply("t", "2"));
}
```

[typealias]: ../../types/alias.md
