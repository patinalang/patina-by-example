# Term

`term` is a special type that represents any BEAM term. Any Patina object can be cast to Term
with `x as term`. `term` has limited use in Patina; you will primarily encounter it as a result
type of external BEAM functions, e.g. defined in Erlang or Elixir.

While individual types can define `From<term>` or `TryFrom<term>` traits, or provide custom coercing
functions, out of the box there are three things you can do with a `term` value:

* `match` it as a basic Erlang type
  (`int`, `float`, `bits`, `atom`, list, map, tuple, `pid`, `port`, `ref`, `fn`),
* use `unsafe { match }` to match it against Patina types based on their runtime representation,
* use `unsafe` `cast_unchecked` to assert - without checking - that it's a specific Patina type.

### `unsafe { match }`

The `unsafe { match }` is unsafe because Patina types and their runtime representations are not
bijective. Consider this snippet:

```rust,ignore
let v = a"nil" as term;

unsafe {
    match v {
        Option::<str>::None(v) => "no integer"
        Option::<int>::None(v) => "no string"
        _ => "invalid value"
    }
}
```

Was `v` intended to be an `Option<str>`, `Option<int>`, an `atom`, or something else entirely?
`unsafe { match }` cannot answer that, as it only matches the runtime representation, which can
be shared by different types.

### `cast_unchecked`

When you know the real type of a `term` in a given situation (for example,
`unicode:characters_to_binary/1` always returns a valid `str`), `cast_unchecked` lets you inform
Patina of the type without adding a runtime check.

```rust,ignore,editable
extern mod erlang;

pub fn main() {
    // External functions return a `term`, but `erlang:length/1` always returns an integer.
    let len = unsafe { erlang::length([1, 2, 3]).cast_unchecked::<int>() };

    println!("{}", len * 2);
}
```

`cast_unchecked` needs an `unsafe` block because, if the `term` isn't really of the asserted type,
the types in the source can mismatch reality with a cascading effect.
When you're not sure, `match` the `term` instead.

```rust,ignore
use Elixir::ExApp::Events;

enum UserEvent {
    Click(int, int),
    Exit
};

fn next_event(seqnum: int, prev_event: UserEvent) -> Result<UserEvent, DecodeError> {
    // External functions can take any type as input, and return `term` as a result.
    let next_event: term = Events.get_next_event(seqnum, prev_event);

    // `term` can be refined to another type using `unsafe_from_term` or `term::try_cast_into`.
    // These two produce equivalent results:
    let next_event_res: Result<UserEvent, DecodeError> = next_event.try_cast_into<UserEvent>();
    let next_event_res: Result<UserEvent, DecodeError> = UserEvent::unsafe_from_term(next_event);

    // Usually, you will write this all in one go.
    // Inside functions returning Result<_, DecodeError>, you will be able to use the ? operator
    // to immediately return the error result. Note that the type here is not `Result<UserEvent>`.
    // More on this later!
    let next_event: UserEvent = Events.get_next_event(seqnum, prev_event).try_cast_into<UserEvent>()?;

    Ok(next_event)
}

pub fn main() {
    match next_event(1, UserEvent::Click(3, 4)) {
        Ok(Click(x, y)) => println!("Clicked x={} y={}", x, y),
        Ok(Exit) => println!("Exited"),
        Err(err) => println!("Failed to decode Elixir result {:?}", err),
    }
}
```

Please note the `unsafe` in `unsafe_from_term`/`try_cast_into` - these functions only verify that
runtime representation matches the type. This can violate invariants of your types, for example
when a runtime tuple `{a"kelvin", -1}` is converted into `Kelvin(int)`. For this reason, you should
prefer implementing `TryFrom<Term>` for your types, and only use `try_cast_into` for low level
plumbing. -->
