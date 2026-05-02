# Unsafe Operations

As an introduction to this section, to borrow from [the official Rust docs][Rust unsafe],
"one should try to minimize the amount of unsafe code in a code base." With that
in mind, let's get started! Unsafe annotations in Patina are used to bypass
protections put in place by the compiler; specifically, there are two primary
things that unsafe is used for:

* calling functions or methods which are `unsafe`
* matching `term` type into Patina types by their runtime representation

### Calling Unsafe Functions

Some functions can be declared as `unsafe`, meaning it is the programmer's
responsibility to ensure correctness instead of the compiler's. One example
of this is `term::try_cast_into`, which will attempt to convert a runtime value
into a Patina type based only on its non-unique runtime representation.

```rust,ignore,editable
struct Celcius(int);
struct Fahrenheit(int);

pub fn main() {
    let degrees_term = Celcius(10) as term;

    unsafe {
        // Celcius(10) and Fahrenheit(10) have the same runtime representation,
        // so `try_cast_into` will succeed.
        // But `degrees_term` wasn't Fahrenheit! Even with some guardrails,
        // this circumvents the type system - which is why it's unsafe.
        let fahrenheit = degrees_term.try_cast_into::<Fahrenheit>().unwrap();
    }
}
```

### Matching `term` by runtime representation

This is different syntax, but is unsafe for the same reason as `try_cast_into` above.

In safe code, `match` and `receive` can only destructure `term`s into BEAM-native types.
When wrapped in `unsafe`, both of these blocks can try to destructure into Patina types by their
runtime representation, similarly to `try_cast_into`:

```rust,ignore,editable
pub fn main() {
    let v = Option::<int>::None as term;

    let safe_destription = match v {
        a"nil" => "the atom nil",
        (a"some", val) => "some value (tagged?)",
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


## See also:

[term][term], [unsafe { match }][unsafe match], [unsafe { receive }][unsafe receive]

[Rust unsafe]: https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html
[term]: primitives/term.md
[unsafe match]: flow_control/match/unsafe_match.md
[unsafe receive]: flow_control/receive/unsafe_receive.md
