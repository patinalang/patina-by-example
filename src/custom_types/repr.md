# Runtime representation

Patina has multiple possible runtime representations for its structs and enum variants,
to adequately model existing usage patterns in the BEAM ecosystem. The same rules apply to both;
the main difference is that an enum's top-level `#[repr(...)]` sets the default for its variants.

You can explicitly set a representation by annotating the struct/variant with a `#[repr(...)]`
attribute. If you don't, Patina will choose an efficient representation for your data, which
can change between language versions. This is equivalent to setting `#[repr(Patina)]`.

If you need a particular representation, it's a good idea to explicitly opt-in via `#[repr(...)]`,
even if it's the default - Patina compiler will help you maintain the invariant.

### Module

Every struct and enum in patina gets its own Erlang module. By default, the module name
is `Elixir.Patina.{fully-qualified definition path}`, but it can be set manually through
`#[module = "mymod"]` attribute. All enum variants share one module.

```rust,ignore,editable
mod my_mod {
    // This struct's module name is `Elixir.Patina.ByExample.MyMod.MyStruct`.
    struct MyStruct;

    // This struct's module name is `my_other_struct`.
    #[module = "my_other_struct"]
    struct MyOtherStruct;
}
```

### Tag

Some structs and variant representations use *tag*. By default, the tag is equal to the `module`
(as above) for `atom` and `tagged` reprs, and to the short variant name for `native_struct`. It can
be set manually through `#[repr(tag="mytag")]`.

```rust,ignore,editable
mod my_mod {
    #[repr(tagged)]
    struct MyStruct(int, int);

    #[repr(tagged, tag = "my_tag")]
    struct MyOtherStruct(int, int);

    // `tag_style` can be set to automatically derive a snake case tag.
    #[repr(tagged, tag_style = "snake_case")]
    struct MySnakeCaseStruct(int, int);

    pub fn main() {
        println!("{:?}", MyStruct(1, 2) as term);
        // (a"Elixir.Patina.ByExample.MyMod.MyStruct", 1, 2)

        println!("{:?}", MyOtherStruct(1, 2) as term)
        // (a"my_tag", 1, 2)

        println!("{:?}", MySnakeCaseStruct(1, 2) as term)
        // (a"my_snake_case_struct", 1, 2)
    }
}
```

### Optimized representation

Unit (`struct Foo;`) and newtype (`struct Foo(int)`) structs/variants are eligible for optimized
representation. Unit structs/variants can be represented by atoms (`#[repr(atom)]`), while
newtypes can be transparently represented by the value they wrap (`#[repr(transparent)]`). These
optimizations are the default for `#[repr(Patina)]`.

## `#[repr(atom)]`

Can only be set on unit structs/variants. Encodes the value as a single atom equal to the tag.
This is the default for unit structs/variants.

```rust,ignore,editable
struct Unit;

#[repr(atom, tag = "custom_name")]
struct CustomNameUnit;

// repr settings on an enum are inherited by variants
#[repr(tag_style = "snake_case")]
enum NotJustUnits {
    VariantA,
    #[repr(tag = "variantB")]
    VariantB,
    VariantC(int),
}

// Explicitly setting `#[repr(atom)]` will ensure that only unit variants
// are ever added to an enum.
#[repr(atom, tag_style = "snake_case")]
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

### Structs

```rust,ignore,editable
#[repr(tagged)]
struct Pair(int, float);

#[repr(tagged, tag = "Elixir.BarBar")]
struct Bar;

pub fn main() {
    println!("{:?}", Pair(42, 3.14) as term);
    // (a"Elixir.Patina.ByExample.Pair", 42, 3.14)

    println!("{:?}", Bar as term);
    // (a"Elixir.BarBar")
}
```

### Enums

```rust,ignore,editable
#[repr(tagged)]
enum TaggedEnum {
    Unit,
    TupleStyle(str),
    StructStyle{x: int, y: int},

    #[repr(atom)]
    UnitWithAtomOverride,

    #[repr(tag = "custom_tag")]
    WithCustomTag(int),
}

pub fn main() {
    use TaggedEnum::*;

    // Unit inherits #[repr(tagged)] from the enum setting.
    println!("{:?}", Unit as term);
    // (a"Elixir.Patina.ByExample.TaggedEnum.Unit")

    // UnitWithAtomOverride overrides the enum repr.
    println!("{:?}", UnitWithAtomOverride as term);
    // a"Elixir.Patina.ByExample.TaggedEnum.UnitWithAtomOverride"

    // Both tuple- and struct-style variants are represented as tagged tuples.
    println!("{:?}", TaggedEnum::TupleStyle("hello") as term);
    // (a"Elixir.Patina.ByExample.TaggedEnum.TupleStyle", "hello")

    println!("{:?}", TaggedEnum::StructStyle{x: 1, y: 2} as term);
    // (a"Elixir.Patina.ByExample.TaggedEnum.StructStyle", 1, 2)

    // You can set a custom tag by specifying it with `#[repr(tag = "...")]`.
    println!("{:?}", TaggedEnum::WithCustomTag(42) as term);
    // (a"custom_tag", 42)
}
```

### Snake Case style

For easier interop with the wider ecosystem, the `repr(tagged)` attribute also takes a `tag_style`
setting that can be set to `snake_case`. When set, the atom tags are the snake case version
of the variant names.

```rust,ignore,editable
#[repr(tagged, tag_style = "snake_case")]
enum SnakeCaseEnum {
    #[repr(atom)]
    UnitVariant,
    StructHTTPStyle{code: int, body: str},
}

pub fn main() {
    println!("{:?}", SnakeCaseEnum::UnitVariant as term);
    // a"unit_variant"

    println!("{:?}", SnakeCaseEnum::StructHTTPStyle{code: 200, body: "hi"} as term);
    // (a"struct_http_style", 200, "hi")
}
```

## Transparent

### Structs

By default, single-element structs are represented in runtime as the element itself.

```rust,ignore,editable
// Omitting repr is equivalent to `#[repr(Patina)]`.
// In a special case of a single-element struct, it's also equivalent to `#[repr(transparent)]`.
struct Celsius(float);

pub fn main() {
    println!("{:?}", Celsius(2.345) as term);
    // 2.345
}
```

### Enums

Like with structs, `#[repr(transparent)]` can only be set for single-element variants.
**The variants explicitly marked with `transparent` have to be uniquely distinguishable
by their runtime shape**, otherwise they will be rejected by the compiler.

Under the default `#[repr(Patina)]`, single-field variants will be transparent by default,
but they'll silently fall back to `native_record` representation if their shapes are not unique.
Thus, adding new variants to an enum can change existing variants' representation!
To guarantee stable representation, use explicit `#[repr(...)]` attributes.

```rust,ignore,editable
#[repr(Elixir)]
struct Point {
    x: int,
    y: int,
}

#[repr(transparent)]
enum TransparentEnum {
    IntVariant(int),
    PointVariant{point: Point},
}

// Compiling this enum will result in an error, as the variants
// are not uniquely distinguishable at runtime:
//
// #[repr(transparent)]
// enum InvalidTransparentEnum {
//     MyName,
//     OrAtom(atom)
// }

pub fn main() {
    use TransparentEnum::*;

    println!("{:?}", IntVariant(200) as term);
    // 200

    println!("{:?}", PointVariant{point: Point{x: 1, y: 2}} as term);
    // #{ a"__struct__" => a"Elixir.Patina.ByExample.Point", a"x" => 1, a"y" => 2 }
}
```

### `repr(transparent)` and generic open shape

Because Patina doesn't monomorphize types, the unique-shape check described above can't be performed
at compilation time for variants that contain a generic type `T`.

Instead, **a transparent variant holding `T` (an "open-shape" variant) will become `tagged` at
runtime** construction if its *value* can overlap any other variant of the enum. There can also be
only *one* transparent open-shape variant per enum - because the compiled code doesn't know
the shape a-priori, it can assign unknown shapes only to a single variant.

Under `#[repr(Patina)]`, the same value rules apply.

```rust,ignore,editable
#[repr(transparent)]
enum TransparentWithTags<T> {
    Untagged(str),

    // `MaybeUntagged(T)` values are represented as themselves, except
    // for values that could be mistaken for another variant.
    // This behaviour has to be opted in with `dynamic_wrap` attribute.
    #[repr(dynamic_wrap, tag = "othertag")]
    MaybeUntagged(T),

    // Individual enum variants can have their own representation set.
    // Tagged(int) will always be tagged with `(a"mytag", _)`.
    #[repr(tagged, tag = "mytag")]
    Tagged(int),
}

pub fn main() {
    use TransparentWithTags::*;

    // A `str` payload could be mistaken for Untagged, so it's wrapped.
    println!("{:?}", MaybeUntagged("hello") as term);
    // (a"othertag", "hello")

    // An int payload cannot be mistaken for anything - it stays transparent.
    println!("{:?}", MaybeUntagged(42) as term);
    // 42

    // The decision is per-value, not per-type. Most terms stay transparent...
    println!("{:?}", MaybeUntagged(a"hi" as term) as term);
    // a"hi"

    // ...but a term that looks like another variant is wrapped, too.
    println!("{:?}", MaybeUntagged((a"othertag", 1) as term) as term);
    // (a"othertag", (a"othertag", 1))
}
```

Only the open-shaped variant can be wrapped in this situation. All other transparent variants
have stable representation that's checked for uniqueness at compilation time.

## Native Records

[Native Records][native record] are another possible representation of structs and enum variants.
Under the default `#[repr(Patina)]`, they are how all non-unit-or-newtype structs and variants
are encoded.

```rust,ignore,editable
#[repr(native_record)]
enum NativeRecordEnum {
    Point(int, int),

    #[repr(tag = "custom_name")]
    WithCustomName(float),
}

// For structs (not enums!) you can reference an existing, external native record.
// The record's existence and fields will be checked on module load.
// Note that `#[repr(record_module = "...")]` does *not* influence the name of the
// module that will hold the struct's implementation.
// For that, use `#[module = "..."]`.
#[repr(native_record, record_module = "external_module", tag = "custom_name")]
struct NativeRecordStruct{
    foo: str,
}

extern mod records;
pub fn main() {
    let point = NativeRecordEnum::Point(1, 2);
    // The field names of a tuple-structs and -variants are atomized integers.
    println!("{:?}", records::get_field_names(point));
    // [a"0", a"1"]

    let rect = NativeRecordStruct{ foo: "bar" };
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
    println!("{:?}", Point{x: 5.2, y: 0.4} as term);
    // #{ a"__struct__" => a"Elixir.Patina.ByExample.Point", a"x" => 5.2, a"y" => 0.4 }
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
    #[repr(tag = "ok")]
    Ok(T),

    #[repr(tag = "error")]
    Err(E),
}

pub fn main() {
    println!("{:?}", Result::<int, str>::Ok(42) as term);
    // (a"ok", 42)

    println!("{:?}", Result::<int, str>::Err("reason") as term);
    // (a"error", "reason")

    // Note that `Result<T, E>` can't represent a bare `:ok` by itself,
    // but we have helpers for that.
    println!("{:?}", Result::<(), str>::Ok(()).into::<compat::BareOkResult>() as term);
    // a"ok"
}
```

### Case study: `Option`

With `#[repr(transparent)]`, Patina's `Option` can cleanly represent most
of the common `t() | nil` combinations:

```rust,ignore,editable
#[repr(transparent)]
pub enum Option<T> {
    #[repr(dynamic_wrap, tag = "some")]
    Some(T),

    #[repr(atom, tag = "nil")]
    None,
}

pub fn main() {
    println!("{:?}", Option::<int>::Some(42) as term);
    // 42

    println!("{:?}", Option::<int>::None as term);
    // a"nil"

    // `Option<atom>` is transparent as long as the atom itself is not `nil`.
    println!("{:?}", Option::<atom>::Some(a"hi") as term);
    // a"hi"
    println!("{:?}", Option::<atom>::Some(a"nil") as term);
    // (a"some", a"nil")

    // Nested options stay distinguishable: only the values that would be
    // mistaken for `None` are wrapped.
    println!("{:?}", Option::<Option<int>>::Some(Option::Some(42)) as term);
    // 42
    println!("{:?}", Option::<Option<int>>::Some(Option::None) as term);
    // (a"some", a"nil")
    println!("{:?}", Option::<Option<int>>::None as term);
    // a"nil"
}
```

### Case study: keyword list

Typing most keyword lists also becomes easy. For example, we could type options
for `erlang:term_to_binary` like this:

```rust,ignore,editable
extern mod erlang;

#[repr(tagged, tag_style = "snake_case")]
pub enum TermToBinaryOpts {
    Compressed(int),
    #[repr(atom)]
    Deterministic,
    MinorVersion(int),
    #[repr(atom)]
    Local,
}

pub fn main() {
    let opts: [TermToBinaryOpts] = [Deterministic, MinorVersion(2)];
    println!("{:?}", opts as term);
    // [a"deterministic", (a"minor_version", 2)]

    println!("{:?}", erlang::term_to_binary(42, opts));
    // <<131, 97, 42>>
}
```

[native record]: https://github.com/erlang/eep/blob/master/eeps/eep-0079.md
