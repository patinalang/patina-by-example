# `unsafe { match }`

Ordinary [`match`][match] can safely destructure `struct`s and distinguish `enum`s, because Patina type
system guarantees that there's a unique mapping between the `match` runtime value and its type.

`term` is an exception: it can still be `match`ed, because it's very useful to do so, but in safe
Patina code we can only destructure it to BEAM-level primitives.

That's where `unsafe { match }` comes in. It uses the same arm syntax as `match`, but it maches
on types' runtime representation instead of source-level identity.

```rust,ignore,editable
pub fn main() {
    let v = Option::<int>::None as term;

    let safe_description = match v {
        a"nil" => "the atom nil",
        (a"some", val) => "some value (tagged?)",
        int(_) => "some integer value (untagged?)",
        val => "some value (untagged?)",
        // Option<str>::None => "string option (none)",
        // ^ERROR - can't safely match by runtime representation!
    };

    let unsafe_description = unsafe {
        match v {
            Option::<str>::None => "an empty string option",
            Option::<int>::None => "an empty integer option",
            a"nil" => "the atom nil",
            _ => "something else",
        }
    };

    println!("{}, {}", safe_description, unsafe_description);
    // "the atom nil, an empty string option"
}
```

`unsafe { match }` is unsafe, because multiple Patina types may lower to the same BEAM value. In the
example above, all three of `Option::<str>::None`, `Option::<int>::None`, and `a"nil"` may share
the same runtime representation. `unsafe { match }` can only see that representation; it cannot
recover which Patina type the value was meant to have.

It's best practice to only use `unsafe { match }` as glue, for interop with external BEAM libraries
and features implemented in other languages.

### See also:

[`unsafe`][unsafe], [`term`][term], and [`unsafe { receive }`][unsafe receive]

[match]: ../match.md
[term]: ../../primitives/term.md
[unsafe]: ../../unsafe.md
[unsafe receive]: ../receive/unsafe_receive.md
