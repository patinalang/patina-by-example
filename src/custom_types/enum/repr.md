# Runtime representation

In Patina, enums have multiple possible runtime representations to adequately model existing usage
patterns in the BEAM ecosystem.

You can explicitly set a representation by annotating the `enum` with a `#[repr(...)]` attribute.
If you don't, Patina compiler will try to use the `transparent` representation if possible,
otherwise falling back to `tagged` (see below).

### Tagged enums

```rust,ignore,editable
#[repr(tagged)]
enum TaggedEnum {
    Unit,
    TupleStyle(str),
    StructStyle{x: int, y: int},

    WithUnitElem((), int),
    WithOneUnitElem(()),

    #[tag(a"my_tag")]
    WithCustomTag,
}

pub fn main() {
    // Unit variants are represented directly as atoms.
    println!("{:?}", TaggedEnum::Unit as term);
    // a"Elixir.Patina.ByExample.TaggedEnum.Unit"

    // Both tuple- and struct-style variants are represented as tagged tuples.
    println!("{:?}", TaggedEnum::TupleStyle("hello") as term)
    // (a"Elixir.Patina.ByExample.TaggedEnum.TupleStyle", "hello")

    println!("{:?}", TaggedEnum::StructStyle{x: 1, y: 2} as term)
    // (a"Elixir.Patina.ByExample.TaggedEnum.StructStyle", 1, 2)

    // Unit elements are omitted in the runtime representation. If all elements
    // of the variant are unit type, the variant is represented directly
    // as an atom.
    println!("{:?}", TaggedEnum::WithUnitElem((), 42) as term)
    // (a"Elixir.Patina.ByExample.TaggedEnum.WithUnitElem", 42)

    println!("{:?}", TaggedEnum::WithOneUnitElem(()) as term)
    // a"Elixir.Patina.ByExample.TaggedEnum.WithOneUnitElem"

    // You can set use a custom tag by specifying it with `tag` attribute.
    println!("{:?}", TaggedEnum::WithCustomTag as term)
    // a"my_tag"
}
```

#### Snake Case style

For easier interop with the wider ecosystem, the `repr(tagged)` attribute also takes a `style`
setting that can be set to `snake_case`. When set, the atom tags are the snake case version
of the variant names.

```rust,ignore,editable
#[repr(tagged, style(snake_case))]
enum SnakeCaseEnum {
    UnitVariant,
    StructHTTPStyle{code: int, body: str}
}

pub fn main() {
    println!("{:?}", SnakeCaseEnum::UnitVariant as term)
    // a"unit_variant"

    println!("{:?}", SnakeCaseEnum::StructHTTPStyle{code: 200, body: "hi"} as term)
    // (a"struct_http_style", 200, "hello")
}
```

### Transparent enums

If your variants can be uniquely distinguished by their runtime shape alone, you can annotate
the enum with `#[repr(transparent)]`. Variants of transparent enums will not be tagged.

```rust,ignore,editable
struct Point {
    x: int,
    y: int,
}

#[repr(transparent)]
enum TransparentEnum {
    Unit,
    TupleStyle(int, str),
    StructStyle{point: Point}

    #[tag(a"forced_tag")]
    ForcedTagVariant(int, str)
}

// Compiling this enum will result in an error, as the variants
// are not uniquely distinguishable at runtime:
//
//   TransparentEnum::MyName ==
//     TransparentEnum::OrAtom(a"Elixir.Patina.ByExample.TransparentEnum.MyName")
//
#[repr(transparent)]
enum InvalidTransparentEnum {
    MyName,
    OrAtom(atom)
}

pub fn main() {
    println!("{:?}", TransparentEnum::Unit as term)
    // a"Elixir.Patina.ByExample.TransparentEnum.Unit"

    println!("{:?}", TransparentEnum::TupleStyle(200, "hello") as term)
    // (200, "hello")

    println!("{:?}", TransparentEnum::StructStyle{point: Point{x: 1, y: 2}} as term)
    // #{ a"__struct__" => a"Elixir.Patina.ByExample.Point", a"x" => 1, a"y" => 2 }

    // ForcedTagVariant is tagged while other variants are transparent.
    // This tag distinguishes it from the `TupleStyle` variant, even though
    // they carry same types
    println!("{:?}", TransparentEnum::ForcedTagVariant(200, "hello") as term)
    // (a"forced_tag", 200, "hello")
}
```


## Consequences for BEAM interop

The rules specified above let Patina directly use enum types to represent
tagged tuples commonly used in BEAM ecosystem.

### Case study: `Result`

Patina's `Result` type is one prominent example of tagged enum use:

```rust,ignore,editable
#[repr(tagged)]
pub enum Result<T, E> {
    #[tag(a"ok")]
    Ok(T),

    #[tag(a"error")]
    Err(E)
}

pub fn main() {
    println!("{:?}    {:?}",
        Result::<int, str>::Ok(42), Result::<int, str>::Err("reason"))
    // (a"ok", 42)    (a"error", "reason")

    // The unit elision rule lets us directly represent `:ok | {:error, reason}`
    println!("{:?}    {:?}",
        Result::<(), str>::Ok(()), Result::<(), str>::Err("reason"))
    // a"ok"    (a"error", "reason")

    // Elixir's `Map.fetch` result `{:ok, val} | :error` can also be represented
    println!("{:?}    {:?}",
        Result::<int, ()>::Ok(42), Result<int, ()>::Err(()))
    // (a"ok", 42)    a"error"
}
```

### Case study: `Option`

Because `transparent` is the default representation where possible, Patina's `Option` can cleanly
represent most of the common `t() | nil` combinations:

```rust,ignore,editable
// when repr is not explicitly set, repr(Patina) is used
pub enum Option<T> {
    #[tag(a"some", forced(false))]
    Some(T),

    #[tag(a"nil")]
    None
}

pub fn main() {
    println!("{:?}    {:?}",
        Option::<int>::Some(42), Option::<int>::None)
    // 42    a"nil"

    // `Option<atom>` cannot be transparent, as `a"nil"` is a valid `Some`
    println!("{:?}    {:?}",
        Option::<atom>::Some(a"hi"), Option::<atom>::None)
    // (a"some", a"hi")    a"nil"

    // Option<Option<int>> also cannot be transparently represented
    println!("{:?}    {:?}    {:?}",
        Option::<Option<int>>::Some(Some(42)),
        Option::<Option<int>>::Some(None),
        Option::<Option<int>>::None)
    // (a"some", 42)    (a"some", a"nil")    a"nil"
}
```

### Case study: keyword list

Typing most keyword lists also becomes easy. For example, we could type options
for `erlang:term_to_binary` like this:

```rust,ignore,editable
extern mod erlang;

#[repr(tagged, style(snake_case))]
pub enum TermToBinaryOpts {
    Compressed(int),
    Deterministic,
    MinorVersion(int),
    Local,
}

pub fn main() {
    let opts: [TermToBinaryOpts] = [Deterministic, MinorVersion(2)]
    println!("{:?}", opts as term)
    // [a"deterministic", (a"minor_version", 2)]

    println!("{:?}", erlang::term_to_binary(val, opts))
    // <<131, 97, 42>>
}
```
