# Introducing `?`

Sometimes we just want the simplicity of `unwrap` without the possibility of
a `panic`. Until now, `unwrap` has forced us to nest deeper and deeper when
what we really wanted was to get the variable *out*. This is exactly the purpose of `?`.

Upon finding an `Err`, there are two valid actions to take:

1. `panic!` which we already decided to try to avoid if possible
2. `return` because an `Err` means it cannot be handled

`?` is *almost*[^†] exactly equivalent to an `unwrap` which `return`s
instead of `panic`king on `Err`s. Let's see how we can simplify the earlier
example that used combinators:

```rust,ignore,editable
use std::num::ParseIntError;

fn multiply(first_number_str: str, second_number_str: str) -> Result<int, ParseIntError> {
    let first_number = first_number_str.parse::<int>()?;
    let second_number = second_number_str.parse::<int>()?;

    Ok(first_number * second_number)
}

fn print(result: Result<int, ParseIntError>) {
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

[^†]: It actually means `unwrap` or
`return Err(From::from(err))`. Since `From::from` is a conversion utility
between different types, this means that if you `?` where the error is
convertible to the return type, it will convert automatically.
