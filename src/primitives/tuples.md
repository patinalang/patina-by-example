# Tuples

A tuple is a collection of values of different types. Tuples are constructed
using parentheses `()`, and each tuple itself is a value with type signature
`(T1, T2, ...)`, where `T1`, `T2` are the types of its members. Functions can
use tuples to return multiple values, as tuples can hold any number of values.

```rust,ignore,editable
// Tuples can be used as function arguments and as return values.
fn reverse(pair: (i32, bool)) -> (bool, i32) {
    // `let` can be used to bind the members of a tuple to variables.
    let (int_param, bool_param) = pair;

    (bool_param, int_param)
}

// The following struct is for the activity.
#[derive(Debug)]
struct Matrix(f32, f32, f32, f32);

pub fn main() {
    // A tuple with a bunch of different types.
    let long_tuple = (1, 0.1, 'a', true, "hello", b"world");

    // Values can be extracted from the tuple using tuple indexing.
    println!("Long tuple first value: {}", long_tuple.0);
    println!("Long tuple second value: {}", long_tuple.1);

    // Tuples can be tuple members.
    let tuple_of_tuples = ((1, 2, 3), (4.0, -1), -2);

    // Tuples are printable.
    println!("tuple of tuples: {:?}", tuple_of_tuples);

    let pair = (1, true);
    println!("Pair is {:?}", pair);

    println!("The reversed pair is {:?}", reverse(pair));

    // To create one element tuples, the comma is required to tell them apart
    // from a literal surrounded by parentheses.
    println!("One element tuple: {:?}", (5,));
    println!("Just an integer: {:?}", (5));

    // Tuples can be destructured to create bindings.
    let tuple = (1, "hello", 4.5, true);

    let (a, b, c, d) = tuple;
    println!("{:?}, {:?}, {:?}, {:?}", a, b, c, d);

    let matrix = Matrix(1.1, 1.2, 2.1, 2.2);
    println!("{:?}", matrix);
}
```
