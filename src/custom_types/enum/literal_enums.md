# Literal enums

Patina's `enum` can be used to enumerate literals, as long as:
* all of the literals are of the same type, and
* all of the literals are unique, and
* all of the enum variants are units.

Literal enum definitions use `=` syntax to provide the literal value.

```rust,ignore,editable
enum AtomEnum {
    Foo = a"hello_foo",
    Bar = a"hello_bar",
}

enum IntEnum {
    Ok = 200,
    NotFound = 404,
}

enum StrEnum {
    Peter = "Peter",
    Paul = "Paul",
}

pub fn main() {
    // literal enums can be safely cast to their underlying type
    println!("{:?}", AtomEnum::Foo as atom)
    // a"hello_foo"

    println!("{:?}", IntEnum::NotFound as int)
    // 404

    println!("{:?}", StrEnum::Peter as str)
    // "Peter"
}
```

Because [all unit variants are represented as atoms][repr], an `enum` comprised solely of unit
variants is also a literal `enum` of type `atom`.

```rust,ignore,editable
enum AllUnits {
    Foo,
    Bar,
    Baz,
}

pub fn main() {
    println!("{:?}", AllUnits::Foo as atom)
    // a"Elixir.Patina.ByExample.AllUnits.Foo"
}
```

[repr]: repr.md
