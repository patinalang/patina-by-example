# Aliasing

The `type` statement can be used to give a new name to an existing type. Types
must have `UpperCamelCase` names, or the compiler will raise a warning.

```rust,ignore,editable
// `NanoSecond`, `Inch`, and `Int` are new names for `int`.
type NanoSecond = int;
type Inch = int;
type Int = int;

pub fn main() {
    // `NanoSecond` = `Inch` = `Int` = `int`.
    let nanoseconds: NanoSecond = 5 as int;
    let inches: Inch = 2 as Int;

    // Note that type aliases *don't* provide any extra type safety, because
    // aliases are *not* new types
    println!("{} nanoseconds + {} inches = {} unit?",
             nanoseconds,
             inches,
             nanoseconds + inches);
}
```

<!-- PATINA TODO: let's put in a good example once we have it -->
The main use of aliases is to reduce boilerplate; for example the `io::Result<T>` type
is an alias for the `Result<T, io::Error>` type.

### See also:

[Attributes](../attribute.md)
