# The `use` declaration

The `use` declaration can be used to bind a full path to a new name, for easier
access. It is often used like this:

```rust,editable,ignore
use pkg::Deeply::Nested::{
    my_first_function,
    my_second_function,
    AndATraitType
};

pub fn main() {
    my_first_function();
}
```

You can use the `as` keyword to bind imports to a different name:

```rust,ignore,editable
// Bind the `deeply::nested::function` path to `other_function`.
use Deeply::Nested::function as other_function;

fn function() {
    println!("called `function()`");
}

mod Deeply {
    pub mod Nested {
        pub fn function() {
            println!("called `Deeply::Nested::function()`");
        }
    }
}

pub fn main() {
    // Easier access to `Deeply::Nested::function`
    other_function();

    println!("Entering block");
    {
        // This is equivalent to `use Deeply::Nested::function as function`.
        // This `function()` will shadow the outer one.
        use pkg::Deeply::Nested::function;

        // `use` bindings have a local scope. In this case, the
        // shadowing of `function()` is only in this block.
        function();

        println!("Leaving block");
    }

    function();
}
```
