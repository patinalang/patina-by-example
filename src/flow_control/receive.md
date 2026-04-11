# receive

`receive` is a [`match`]-like expression over incoming mailbox values.

Like BEAM `receive`, it scans the mailbox from oldest to newest, consumes the first message that
matches one of its arms, and leaves all non-matching messages in the mailbox. If there is no message
that matches any of the arms, `receive` blocks the current process until one arrives, or until
a timeout is hit, whichever comes first.

Aside from literals and destructuring, Patina lets you match whole BEAM primitive classes
in a single arm. For example, `int(n)` matches any integer and binds it to `n`, while `atom(a)`
matches any atom and binds it to `a`.

```rust,ignore,editable
pub fn main() {
    receive {
        int(n) => println!("received integer {:?}", n),
        // like in `match`, literals can also be used
        a"ping" => println!("received ping"),
        atom(a) => println!("received atom {:?}", a),
    } timeout Duration::from_seconds(5) {
        println!("timeout (5 seconds with no matching message)")
    }

    receive {
        val => println!("received any message {:?}", val),
    }
}
```

The `receive` arms work the same way they do in ordinary `match`:

- patterns narrow the matched message
- the value of the whole `receive` expression is the value of the chosen arm
- every `catch` arm, and the `timeout` block if present, must all have the same result type.

[`match`]: match.md
