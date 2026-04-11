# try/catch/after

Patina code strongly prefers handling errors with `Result<T, E>`, but it lives within a wider
BEAM environment, where process errors are reported with exception-like signals. Similarly, many
existing BEAM libraries leverage exceptions for their error handling, or thrown values for flow
control.

The `try`/`catch`/`after` construct can be used to interact with these throwables in Patina.

If the `try` block completes normally, its value becomes the value of the whole expression. If it
fails with a catchable outcome, the `catch` arms may handle it. The `after` block is for cleanup and
runs when the `try` expression finishes, whether it completed normally or not.

As with `match` and `receive`, the `try` block and every `catch` arm must all have the same result
type. Unlike `match`, but similarly to `receive`, `catch` does not have to cover the whole set
of possibilities - anything unmatched will just pass through to the parent `catch` block, if any.

```rust,ignore,editable
fn open_resource() -> atom {
    a"ok"
}

pub fn main() {
    try {
        open_resource()
    } catch {
        Panic(message) => println!("panic! {}", message),
        ArgumentError(message) => println("badarg! {}", message),
        ElixirError(a"Elixir.ArithmeticError", _) => println!("Elixir ArithmeticError"),
        Error(reason) => println!("other error: {:?}", reason),
        // `trace` can be used to bind the throwable's stacktrace to a variable
        Throw(value) trace st => println!("thrown: {:?} from {:?}", value, st),
        Exit(reason) => println!("exit: {:?}", reason),
        _ => a"unknown",
    } after {
        println!("cleaning up");
    };
}
```

## Catchable classes

A `catch` arm can destructure the following throwables:

- `Panic(str)` - Panics are Patina's take on exceptions. A Panic *can* be ignored (through catch),
  but it's only supposed to be used for unrecoverable errors, to be logged and fixed
  by the programmer;
- `ArgumentError(str)` - due to their ubiquity in the wider ecosystem, both Erlang's `badarg`
  and Elixir's `ArgumentError` get converted to a separate catchable class
- `ElixirError(atom, #{atom => term})`
- `Error(term)` - Error class also includes panics, badargs and Elixir exceptions - match them first
  if you need to handle them in a special way
- `Throw(term)`
- `Exit(term)` - BEAM Process exit signal that propagates over process links

### See also:

[`Result`](../error/result.md), [`panic`](../error/panic.md), and [`flow of control`](../flow_control.md)
