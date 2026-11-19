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
        open_resource();
    } catch {
        error(a"badarg") =>
            println!("badarg!"),

        error(reason @ #{a"__struct__" => a"Elixir.MyElixirException", ..}) =>
            println!("An Elixir exception {:?}", reason),

        // a second pattern after the comma binds the stacktrace
        error(reason), stacktrace =>
            println!("other error: {:?} from {:?}", reason, stacktrace),

        throw(value) =>
            println!("thrown: {:?}", value),

        exit(reason) =>
            println!("exit: {:?}", reason),
    } after {
        println!("cleaning up");
    };
}
```

## Catchable classes

A `catch` arm can destructure the following throwables:

- `error(reason)` / `error(reason), stacktrace` - the BEAM error class. Erlang's `badarg` is
  `error(a"badarg")`, and an Elixir exception is a struct map with `__exception__` set to `true`,
  e.g. `error(#{a"__struct__" => a"Elixir.ArgumentError", ..})`;

- `throw(value)` / `throw(value), stacktrace` - thrown values;

- `exit(reason)` / `exit(reason), stacktrace` - BEAM process exit signals that propagate over
  process links.

- `any(class, value)` / `any(class, value), stacktrace` - any BEAM exception class.
  `class` gets bound to `error`, `throw`, or `exit`.

## Matching on the caught value

The value caught by each class is a [`term`][term], so it can be destructured
like in any other [`match`][match] on a `term`:

```rust,ignore,editable
extern mod erlang;

pub fn main() {
    let value = try {
        erlang::error((a"badmatch", 42));
        0
    } catch {
        // Match a `badmatch` tuple with an `int` value.
        error((a"badmatch", int(n))) => n,
    };

    println!("{}", value);
}
```

### See also:

[`Result`](../error/result.md), [`panic`](../error/panic.md), and [`flow of control`](../flow_control.md)

[term]: ../primitives/term.md
[match]: match.md
