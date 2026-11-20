# Dynamic errors

A way to write simple code while preserving the original errors is to return them with [`dyn`][dyn].
The drawback is that the underlying error type is only known at runtime and not statically determined.

The stdlib helps in dynamically returning our errors by defining an `std::error::Error` trait.
It's already implemented for all standard errors.

```rust,ignore,editable
use std::error;
use std::fmt;

// Change the alias to use `dyn error::Error`.
type Result<T> = std::result::Result<T, dyn error::Error>;

#[derive(Debug)]
struct EmptyList;

impl fmt::Display for EmptyList {
    fn fmt(self, f: fmt::Formatter) -> fmt::Result {
        write!(f, "invalid first item to double")
    }
}

impl error::Error for EmptyList {}

fn double_first(list: [str]) -> Result<int> {
    list.first()
        .ok_or_else(|| EmptyList)
        .and_then(|s| {
            s.parse::<int>()
                .map(|i| 2 * i)
        })
}

fn print(result: Result<int>) {
    match result {
        Ok(n) => println!("The first doubled is {}", n),
        Err(e) => println!("Error: {}", e),
    }
}

pub fn main() {
    let numbers = ["42", "93", "18"];
    let empty = [];
    let strings = ["tofu", "93", "18"];

    print(double_first(numbers));
    print(double_first(empty));
    print(double_first(strings));
}
```

[dyn]: ../../trait/dyn.md
