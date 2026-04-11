# Overload

Macros can be overloaded to accept different combinations of arguments.
In that regard, `macro_rules!` can work similarly to a match block:

```rust,ignore,editable
// `test!` will compare `$left` and `$right`
// in different ways depending on how you invoke it:
macro_rules! test {
    // Arguments don't need to be separated by a comma.
    // Any template can be used!
    ($left:expr; and $right:expr) => {
        println!("{:?} and {:?} is {:?}",
                 stringify!($left),
                 stringify!($right),
                 $left && $right)
    };
    // ^ each arm must end with a semicolon.
    ($left:expr; or $right:expr) => {
        println!("{:?} or {:?} is {:?}",
                 stringify!($left),
                 stringify!($right),
                 $left || $right)
    };
}

pub fn main() {
    test!(1 + 1 == 2; and 2 * 2 == 4);
    test!(true; or false);
}
```
