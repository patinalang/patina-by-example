# Attributes

An attribute is metadata applied to some module or item. This metadata
can be used to/for:

* set BEAM runtime representation for [structs][structs repr] and [enums][enums repr]
* [set BEAM module name for Patina modules][mod visibility]
* [set BEAM module names for Patina structs and enums][struct visibility]
* [attribute like macros][macros]

Attributes look like `#[outer_attribute]` or `#![inner_attribute]`,
with the difference between them being where they apply.

* `#[outer_attribute]` applies to the item immediately
  following it. Some examples of items are: a function, a module
  declaration, a constant, a structure, an enum. Here is an example
  where attribute `#[derive(Debug)]` applies to the struct
  `Rectangle`:

  ```rust,ignore
  #[derive(Debug)]
  struct Rectangle {
      width: int,
      height: int,
  }
  ```

* `#![inner_attribute]` applies to the enclosing item (typically a
  module). In other words, this attribute is interpreted as
  applying to the entire scope in which it's placed. Here is an example
  where `#![beam_module("Elixir.MyMod")]` applies to the enclosing module:

  ```rust,ignore
  mod MyMod {
    #![beam_module("Elixir.MyMod")]

    fn inner() {
      println!("inner called!");
    }
  }

  pub fn main() {
    MyMod::inner();
  }
  ```

Attributes can take arguments with different syntaxes:

* `#[attribute = "value"]`
* `#[attribute(key = "value")]`
* `#[attribute(value)]`

Attributes can have multiple values and can be separated over multiple lines, too:

```rust,ignore
#[attribute(value, value2)]


#[attribute(value, value2, value3,
            value4, value5)]
```

[structs repr]: custom_types/structs/repr.md
[enums repr]: custom_types/enum/repr.md
[mod visibility]: mod/visibility.md
[struct visibility]: mod/struct_visibility.md
[macros]: macros.md
