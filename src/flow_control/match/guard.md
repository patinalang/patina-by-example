# Guards

A `match` *guard* can be added to filter the arm.
Only [Erlang guard expressions][guard expressions] are allowed in the `if` guards.

<!-- Patina TODO: Patina-native guard expression set -->

```rust,ignore,editable
#[allow(dead_code)]
enum Temperature {
    Celsius(int),
    Fahrenheit(int),
}

pub fn main() {
    let temperature = Temperature::Celsius(35);
    // ^ TODO try different values for `temperature`

    match temperature {
        Temperature::Celsius(t) if t > 30 => println!("{}C is above 30 Celsius", t),
        // The `if condition` part ^ is a guard
        Temperature::Celsius(t) => println!("{}C is equal to or below 30 Celsius", t),

        Temperature::Fahrenheit(t) if t > 86 => println!("{}F is above 86 Fahrenheit", t),
        Temperature::Fahrenheit(t) => println!("{}F is equal to or below 86 Fahrenheit", t),
    }
}
```

Note that the compiler won't take guard conditions into account when checking
if all patterns are covered by the match expression.

```rust,editable,ignore
pub fn main() {
    let number: int = 4;

    match number {
        i if i == 0 => println!("Zero"),
        i if i > 0 => println!("Greater than zero"),
        // _ => unreachable!("Should never happen."),
        // TODO ^ uncomment to fix compilation
    }
}
```

### See also:

[Tuples](../../primitives/tuples.md)
[Enums](../../custom_types/enum.md)

[guard expressions]: https://www.erlang.org/doc/system/expressions.html#guard-expressions
