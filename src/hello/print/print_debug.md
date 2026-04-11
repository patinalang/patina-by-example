# Debug

All types which want to use `Std::fmt` formatting `traits` require an
implementation to be printable. Automatic implementations are only provided
for types such as in the `std` library. All others *must* be manually
implemented somehow.

The `Fmt::Debug` `trait` makes this very straightforward. *All* types can
`derive` (automatically create) the `Fmt::Debug` implementation. This is
not true for `Fmt::Display` which must be manually implemented.

```rust,ignore
// This structure cannot be printed either with `Fmt::Display` or
// with `Fmt::Debug`.
struct UnPrintable(i32);

// The `derive` attribute automatically creates the implementation
// required to make this `struct` printable with `Fmt::Debug`.
#[derive(Debug)]
struct DebugPrintable(i32);
```

All `std` library types are automatically printable with `{:?}` too:

```rust,ignore,editable
// Derive the `Fmt::Debug` implementation for `Structure`. `Structure`
// is a structure which contains a single `i32`.
#[derive(Debug)]
struct Structure(i32);

// Put a `Structure` inside of the structure `Deep`. Make it printable
// also.
#[derive(Debug)]
struct Deep(Structure);

pub fn main() {
    // Printing with `{:?}` is similar to with `{}`.
    println!("{:?} months in a year.", 12);
    println!("{1:?} {0:?} is the {actor:?} name.",
             "Slater",
             "Christian",
             actor="actor's");

    // `Structure` is printable!
    println!("Now {:?} will print!", Structure(3));

    // The problem with `derive` is there is no control over how
    // the results look. What if I want this to just show a `7`?
    println!("Now {:?} will print!", Deep(Structure(7)));
}
```

So `Fmt::Debug` definitely makes this printable but sacrifices some elegance.
Rust also provides "pretty printing" with `{:#?}`.

```rust,ignore,editable
#[derive(Debug)]
struct Person<'a> {
    name: &'a str,
    age: u8
}

pub fn main() {
    let name = "Peter";
    let age = 27;
    let peter = Person { name, age };

    // Pretty print
    println!("{:#?}", peter);
}
```

One can manually implement `Fmt::Display` to control the display.

### See also:

[`attributes`][attributes], [`derive`][derive], [`Std::fmt`][fmt],
and [`struct`][structs]

[attributes]: https://doc.rust-lang.org/reference/attributes.html
[derive]: ../../trait/derive.md
[fmt]: https://doc.rust-lang.org/std/fmt/
[structs]: ../../custom_types/structs.md
