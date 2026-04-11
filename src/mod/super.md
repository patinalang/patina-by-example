# `super` and `self`

The `super` and `self` keywords can be used in the path to remove ambiguity
when accessing items and to prevent unnecessary hardcoding of paths.

```rust,ignore,editable
fn function() {
    println!("called `function()`");
}

mod Cool {
    pub fn function() {
        println!("called `Cool::function()`");
    }
}

mod My {
    fn function() {
        println!("called `My::function()`");
    }

    mod Cool {
        pub fn function() {
            println!("called `My::Cool::function()`");
        }
    }

    pub fn indirect_call() {
        // Let's access all the functions named `function` from this scope!
        print!("called `My::indirect_call()`, that\n> ");

        // The `self` keyword refers to the current module scope - in this case `My`.
        // Calling `self::function()` and calling `function()` directly both give
        // the same result, because they refer to the same function.
        self::function();
        function();

        // We can also use `self` to access another module inside `My`:
        self::Cool::function();

        // The `super` keyword refers to the parent scope (outside the `My` module).
        super::function();

        // This will bind to the `Cool::function` in the *pkg* scope.
        // In this case the pkg scope is the outermost scope.
        {
            use pkg::Cool::function as root_function;
            root_function();
        }
    }
}

pub fn main() {
    My::indirect_call();
}
```
