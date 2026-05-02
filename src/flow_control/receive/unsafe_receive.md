# `unsafe receive`

`unsafe { receive }` is to [`receive`][receive] what [`unsafe { match }`][unsafe match]
is to [`match`][match].

Like `receive`, it scans the mailbox using normal selective-receive rules. Unlike `receive`, it can
match not only BEAM-level primitives such as `int(n)` and `atom(a)`, but also Patina types based
only on their runtime representation.

```rust,ignore,editable
pub fn main() {
    unsafe {
        receive {
            int(n) => println!("got integer {:?}", n),
            atom(a) => println!("got atom {:?}", a),
            Option::<str>::None => println!("got an empty string option"),
            Option::<int>::None => println!("got an empty integer option"),
            _ => println!("got something else"),
        } timeout Duration::from_seconds(5) {
            println!("timeout")
        }
    }
}
```

`unsafe { receive }` is unsafe for the same reason [`unsafe { match }`][unsafe match] is unsafe:
different Patina types can lower to the same BEAM value. The message matched above might have been
intended as `Option::<str>::None`, `Option::<int>::None`, an ordinary atom,
or something else entirely.

It's best practice to only use `unsafe { receive }` as a glue, for interop with BEAM libraries
and features implemented in other languages.

### See also:

[`unsafe`][unsafe], [`term`][term], and [`unsafe { match }`][unsafe match]

[match]: ../match.md
[term]: ../../primitives/term.md
[unsafe]: ../../unsafe.md
[unsafe match]: ../match/unsafe_match.md
[receive]: ../receive.md
