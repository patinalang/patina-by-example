# Extern calls

In Patina, all calls to functions are *local* by default - if at all possible.
This means that the Erlang-level calls to functions defined in the same Erlang module do not use
[fully-qualified module path][erl expressions], even if the path is fully-qualified in Patina source
code.

However, in some advanced usage we want to explicitly refer to another function using
its fully-qualified path. For these cases, you can use `#[extern_call]` attribute on the call
expression.

```rust,ignore,editable
fn priv_function() -> int {
    42
}

pub fn public_function() -> int {
    42
}

pub fn main() {
    // Local call: `public_function()`
    public_function();

    // Qualified, but still a local call: `public_function()`
    self::public_function();

    // Remote call: `'Elixir.Patina.MyPkg':public_function()`
    #[extern_call] public_function();

    // Always a local call: `priv_function` is not exported!
    // Prepending `#[extern_call]` would result in a compilation error.
    priv_function();
}
```

[erl expressions]: https://www.erlang.org/doc/system/expressions.html#function-calls
