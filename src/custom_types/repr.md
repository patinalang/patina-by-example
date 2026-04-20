# Runtime representation

Patina has multiple possible runtime representations for its structs and enum variants,
to adequately model existing usage patterns in the BEAM ecosystem. The same rules apply to both;
the main difference is that an enum's top-level `#[repr]` sets the default for its variants.

Variants can be further configured with their own `#[repr(...)]` attribute, setting per-variant
properties like `name`, or changing the representation altogether.

You can explicitly set a representation by annotating the struct or enum with a `#[repr(...)]`
attribute. If you don't, Patina will choose an efficient representation for your data, that
can change between language versions. This is equivalent to setting `#[repr(Patina)]`.

If you need a particular representation, it's a good idea to explicitly opt-in via `repr(...)`,
even if it's the default - Patina compiler will help you maintain the invariant.

## Atom

Unit structs and variants are always encoded directly as their atom names, regardless
of the `#[repr(...)]` setting on the enum.

```rust,ignore,editable
struct Unit;

#[repr(atom, name = "custom_name")]
struct CustomNameUnit;

// `style` can be used to change the style of atom tags assigned by Patina
#[repr(style = snake_case)]
enum NotJustUnits {
    VariantA,
    #[repr(name = "variantB")]
    VariantB,
    VariantC(int)
}

// Explicitly setting `#[repr(atom)]` will ensure that only unit variants
// are ever added to an enum.
#[repr(atom, style = snake_case)]
enum OnlyUnits {
    VariantA,
    VariantB,
    VariantC,
}

pub fn main() {
    println!("{:?}", Unit as term);
    // a"Elixir.Patina.ByExample.Unit"

    println!("{:?}", CustomNameUnit as term);
    // a"custom_name"

    println!("{:?}", NotJustUnits::VariantA as term);
    // a"variant_a"

    println!("{:?}", NotJustUnits::VariantB as term);
    // a"variantB"
}
```

## Tagged

Encoded as tagged tuples, where the first element is the atom uniquely identifying the struct
or variant name, and the following tuple fields are the fields. This is similar to how Erlang's
records are modeled.

In the `tagged` representation, unit type is omitted as a struct field, potentially devolving
down to a single atom representation when all fields are units.

### Structs

```rust,ignore,editable
#[repr(tagged)]
struct Pair(int, float);

#[repr(tagged, name = "Elixir.BarBar")]
struct Bar{ x: int, y: int, z: () }

pub fn main() {
    println!("{:?}", Pair(42, 3.14) as term);
    // (a"Elixir.Patina.ByExample.Pair", 42, 3.14)

    // z is omitted from the representation
    println!("{:?}", Bar(x: 1, y: 2, z: ()) as term);
    // (a"Elixir.BarBar", 1, 2)
}
```

### Enums

```rust,ignore,editable
#[repr(tagged)]
enum TaggedEnum {
    Unit,
    TupleStyle(str),
    StructStyle{x: int, y: int},

    WithUnitElem((), int),
    WithOneUnitElem(()),

    #[repr(name = "custom_tag")]
    WithCustomTag(int),
}

pub fn main() {
    // In every representation, unit variants with are always encoded
    // directly as atoms.
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

    // You can set a custom tag by specifying it with `#[repr(name = "...")]`.
    println!("{:?}", TaggedEnum::WithCustomTag(42) as term)
    // (a"my_tag", 42)
}
```

### Snake Case style

For easier interop with the wider ecosystem, the `repr(tagged)` attribute also takes a `style`
setting that can be set to `snake_case`. When set, the atom tags are the snake case version
of the variant names.

```rust,ignore,editable
#[repr(tagged, style = snake_case)]
enum SnakeCaseEnum {
    UnitVariant,
    StructHTTPStyle{code: int, body: str}
}

pub fn main() {
    println!("{:?}", SnakeCaseEnum::UnitVariant as term)
    // a"unit_variant"

    println!("{:?}", SnakeCaseEnum::StructHTTPStyle{code: 200, body: "hi"} as term)
    // (a"struct_http_style", 200, "hi")
}
```

## Transparent

### Structs

By default, single-element structs are represented in runtime as the element itself.

```rust,ignore,editable
// Omitting repr is equivalent to `#[repr(Patina)]`.
// In a special case of a single-element struct, it's also equivalent to `#[repr(transparent)]`.
struct Celcius(degrees: float);

pub fn main() {
    println!("{:?}", Celcius(2.345) as term);
    // 2.345
}
```

### Enums

If your variants can be uniquely distinguished by their runtime shape alone, you can annotate
the enum with `#[repr(transparent)]`. Variants of transparent enums will not be tagged unless
explicitly forced.

```rust,ignore,editable
#[repr(Elixir)]
struct Point {
    x: int,
    y: int,
}

#[repr(transparent)]
enum TransparentEnum {
    Unit,
    TupleStyle(int, str),
    StructStyle{point: Point}
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
}
```

### Fallback

```rust,ignore,editable
// repr(transparent) can also be given a fallback, if the type cannot
// be transparently represented. This is important for generic types,
// as runtime representation will differ between concrete `T`s.
#[repr(transparent, fallback = tagged)]
enum TransparentWithTags<T> {
    Untagged(str),

    // `MaybeTagged(T)` will only be tagged if T representation conflicts with
    // `Untagged(str)` above.
    #[repr(name = "othertag")]
    MaybeUntagged(T)

    // Individual enum variants can have their own representation set.
    // Tagged(int) will always be tagged with `(a"mytag", _)
    #[repr(tagged, name = "mytag")]
    Tagged(int),
}
```

Native records (shown below) can also be used as a fallback
via `#[repr(transparent, fallback = native_record)]`.

## Native Records

[Native Records][native record] are another possible representation of structs and enum variants.

```rust,ignore,editable
#[repr(native_record)]
enum NativeRecordEnum {
    Point(int, int),
    Rectangle(upper_left: Point, lower_right: Point),

    #[repr(name = "custom_name")]
    WithCustomName(float),
}

// For structs (not enums!) you can reference an existing, external native record.
// The record's existence and fields will be checked on module load.
#[repr(native_record, module = "external_module", name = "custom_name")]
enum NativeRecordStruct{
    foo: str,
}

extern mod records;

pub fn main() {
    let point = Point(1, 2);
    // The field names of a tuple-structs and -variants are atomized integers.
    println!("{:?}", records::get_field_names(point));
    // [a"0", a"1"]

    let rect = NativeRecordStruct(upper_left: point, lower_right: point);
    println!("{:?}", records::get_field_names(rect));
    // [a"foo"]
}
```

## Elixir-style structs

Structs only. These are encoded as fully Elixir-compatible structs, with atom fields,
a `__struct__: module` field, and a module that implements `__struct__/0`, `__struct__/1`
and `__info__(:struct)` callbacks.

```rust,ignore,editable
#[repr(Elixir)]
struct Point {
    x: float,
    y: float,
}

pub fn main() {
    println!("{:?}", Point{x: 1, y: 2} as term);
    // #{a"__struct__" => a"Elixir.Patina.ByExample.Point", a"x" => 5.2, a"y" => 0.4}
}
```

## Consequences for BEAM interop

The rules specified above let Patina directly use enum types to represent tagged tuples commonly
used in the BEAM ecosystem.

### Case study: `Result`

Patina's `Result` type is one prominent example of tagged enum use:

```rust,ignore,editable
#[repr(tagged)]
pub enum Result<T, E> {
    #[repr(name = "ok")]
    Ok(T),

    #[repr(name = "error")]
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

With `#[repr(transparent, fallback = tagged)]`, Patina's `Option` can cleanly represent most
of the common `t() | nil` combinations:

```rust,ignore,editable
#[repr(transparent, fallback = tagged)]
pub enum Option<T> {
    #[repr(name = "some")]
    Some(T),

    #[repr(name = "nil")]
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

#[repr(tagged, style = snake_case)]
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

[native record]: https://github.com/erlang/eep/blob/master/eeps/eep-0079.md
