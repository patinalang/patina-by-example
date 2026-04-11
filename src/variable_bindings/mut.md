# Mutability

Variable bindings are immutable by default, but this can be overridden using
the `mut` modifier.

Note that **this does not mean the values themselves are mutable!** Like other
languages on BEAM VM, Patina's values are immutable; the `mut` modifier only
allows to assign a *new value* to an existing variable.

```rust,editable,ignore
pub fn main() {
    let _immutable_binding = 1;
    let mut mutable_binding = 1;

    println!("Before mutating the binding: {}", mutable_binding);

    // Ok
    mutable_binding += 1;

    println!("After mutating the binding: {}", mutable_binding);

    // Error! Cannot assign a new value to an immutable variable
    _immutable_binding += 1;

    // Inside closures (lambda functions) and `try` blocks, `let mut` variables
    // defined in outer scopes lose their `mut` modifier
    let my_fun = || {
        // Error! Cannot assign a new value to a variable defined outside closure
        mutable_binding += 1;

        // You must define a new variable to modify it inside your closure
        let mut inner_mutable_binding = mutable_binding;
        inner_mutable_binding += 1;
        println("{} == {} + 1", inner_mutable_binding, mutable_binding);
    };
}
```
