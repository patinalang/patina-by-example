# Diverging functions

Diverging functions never return. They are marked using `!`, which is an empty type.

```rust,ignore
fn foo() -> ! {
    panic!("This call never returns.");
}
```

As opposed to all the other types, this one cannot be instantiated, because the
set of all possible values this type can have is empty. Note that, it is
different from the `()` type, which has exactly one possible value.

For example, this function returns as usual, although there is no information
in the return value.

```rust,ignore
fn some_fn() {
    ()
}

pub fn main() {
    let _a: () = some_fn();
    println!("This function returns and you can see this line.");
}
```

As opposed to this function, which will never return the control back to the caller.

```rust,ignore
#![feature(never_type)]

fn some_fn() -> ! {
    panic!("This call never returns.");
}

pub fn main() {
    let _a = some_fn();
    println!("You will never see this line!");
}
```

Although this might seem like an abstract concept, it is actually very useful and
often handy. The main advantage of this type is that it can be cast to any other
type, making it versatile in situations where an exact type is required, such as
in match branches. This flexibility allows us to write code like this:

```rust,ignore
pub fn main() {
    fn sum_odd_numbers(up_to: int) -> int {
        let mut acc = 0;
        for i in 0..up_to {
            // Notice that the return type of this match expression must be int
            // because of the type of the "addition" variable.
            let addition: int = match i%2 == 1 {
                // The "i" variable is of type int, which is perfectly fine.
                true => i,
                // On the other hand, the "continue" expression does not return
                // int, but it is still fine, because it never returns and therefore
                // does not violate the type requirements of the match expression.
                false => continue,
            };
            acc += addition;
        }
        acc
    }
    println!("Sum of odd numbers up to 9 (excluding): {}", sum_odd_numbers(9));
}
```

It is also the return type of functions that loop forever (e.g. `loop {}`) like
network servers or functions that terminate the process (e.g. `exit()`).
