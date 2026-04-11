# Maps

A map is an unordered collection of unique key -> value mappings. Maps are created
using `#{k => v, ...}` syntax, with `#{}` being an empty map.

Map keys can be booleans, integers, strings, or any other type. Every term on BEAM VM
is hashable, and that hash is used for BEAM maps as well.

<!-- // PATINA TODO: pin down the iteration story. We want Elixir's Enumerable to be supported -->

```rust,ignore,editable
fn call(number: str) -> str {
    match number {
        "798-1364" => "We're sorry, the call cannot be completed as dialed.
            Please hang up and try again.",
        "645-7689" => "Hello, this is Mr. Awesome's Pizza. My name is Fred.
            What can I get for you today?",
        _ => "Hi! Who is this again?"
    }
}

pub fn main() {
    let mut contacts = #{
        contacts.insert("Daniel", "798-1364"),
        contacts.insert("Ashley", "645-7689"),
        contacts.insert("Katie", "435-8291"),
        contacts.insert("Robert", "956-1745"),
    };

    // Returns Option<str>
    match contacts.get("Daniel") {
        Some(number) => println!("Calling Daniel: {}", call(number)),
        _ => println!("Don't have Daniel's number."),
    }

    // `.insert()` returns an updated map.
    // The value must be used, otherwise it will emit a compilation warning.
    contacts = contacts.insert("Daniel", "164-6743");

    match contacts.get("Ashley") {
        Some(number) => println!("Calling Ashley: {}", call(number)),
        _ => println!("Don't have Ashley's number."),
    }

    contacts = contacts.remove("Ashley");

    // Like structs, maps can also be updated using the special `..` syntax
    contacts = #{"Laura" => "132-8372", ..contacts};

    // `for` will iterate over the (key, value) pairs in arbitrary order.
    for (contact, number) in contacts {
        println!("Calling {}: {}", contact, call(number));
    }
}
```
