# Constants

Patina has one type of constant, `const`, which can be declared in any scope
including global. It requires an explicit type annotation.

A constant is by default only accessible in the current module.
`pub const` can be used to allow access from other modules.

```rust,editable,ignore
// Globals are declared outside all other scopes.
const THRESHOLD: int = 10;

fn is_big(n: int) -> bool {
    // Access constant in some function
    n > THRESHOLD
}

pub fn main() {
    let n = 16;

    println!("The threshold is {}", THRESHOLD);
    println!("{} is {}", n, if is_big(n) { "big" } else { "small" });

    // Error! Cannot modify a `const`.
    THRESHOLD = 5;
}
```

### Runtime representation

`pub const` will be persisted in the Erlang Abstract Format. It's roughly equivalent
to Elixir's persistent attributes, and will show up in runtime in `__info__(:attributes)`
callback for the module that defines them.
