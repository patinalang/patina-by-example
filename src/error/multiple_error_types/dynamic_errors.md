# Dynamic errors

A way to write simple code while preserving the original errors is to return them with [`dyn`][dyn].
The drawback is that the underlying error type is only known at runtime and not statically determined.

The stdlib helps in dynamically returning our errors by defining an `Std::Error::Error` trait.
It's already implemented for all standard errors.

```rust,ignore,editable
use Std::Error;
use Std::Fmt;

// Change the alias to use `dyn Error::Error`.
type Result<T> = Std::Result::Result<T, dyn Error::Error>;

#[derive(Debug)]
struct EmptyList;

impl Fmt::Display for EmptyList {
    fn fmt(self, f: Fmt::Formatter) -> Fmt::Result {
        write!(f, "invalid first item to double")
    }
}

impl Error::Error for EmptyList {}

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
