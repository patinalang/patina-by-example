# Defining an error type

Sometimes it simplifies the code to mask all of the different errors with a
single type of error.  We'll show this with a custom error.

Patina allows us to define our own error types. In general, a "good" error type:

* Represents different errors with the same type
* Presents nice error messages to the user
* Is easy to compare with other types
  * Good: `Err(EmptyList)`
  * Bad: `Err("Please use a list with at least one element")`
* Can hold information about the error
  * Good: `Err(BadChar(c, position))`
  * Bad: `Err("+ cannot be used here")`
* Composes well with other errors

```rust,ignore,editable
use Std::Fmt;

type Result<T> = Std::Result::Result<T, DoubleError>;

// Define our error types. These may be customized for our error handling cases.
// Now we will be able to write our own errors, defer to an underlying error
// implementation, or do something in between.
#[derive(Debug)]
struct DoubleError;

// Generation of an error is completely separate from how it is displayed.
// There's no need to be concerned about cluttering complex logic with the display style.
//
// Note that we don't store any extra info about the errors. This means we can't state
// which string failed to parse without modifying our types to carry that information.
impl Fmt::Display for DoubleError {
    fn fmt(self, f: Fmt::Formatter) -> Fmt::Result {
        write!(f, "invalid first item to double")
    }
}

fn double_first(list: [str]) -> Result<int> {
    list.first()
        // Change the error to our new type.
        .ok_or(DoubleError)
        .and_then(|s| {
            s.parse::<int>()
                // Update to the new error type here also.
                .map_err(|_| DoubleError)
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
