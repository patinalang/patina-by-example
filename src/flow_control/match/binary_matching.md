# Binary matching

A `match` can destructure `str`ings, `bin`aries, and `bits`trings directly.

Like in [Erlang's bit syntax][erlang bit syntax], binaries are matched in *segments*.
A binary pattern only matches if every segment matches, and the whole value is consumed.

```rust,ignore,editable
pub fn main() {
    let payload = <<1, 17, 42, 99>>;

    match payload {
        <<prefix: int(16), tag: int(8), rest @ ..>> =>
            println!("prefix = {}, tag = {}, rest = {:?}", prefix, tag, rest),

        _ => println!("did not match"),
    }
}
```

Like all `match` patterns, binary patterns can bind values to names, ignore parts with `_`, and be
combined with guards when you need extra checks after destructuring.

## Segments

There can be zero or more segments in a binary match arm. Each segment can have a type
annotation describing how it is matched. An example segment is of the form
`var: int(size=^x, signed, little, unit=8)`, where:

- the segment's type follows a colon: one of `int`, `float`, `char`, `str`, `bin` or `bits`

- `size` can be 0 or more, and can be given through a [pinned][bound variables] variable,
  like `size=^x` - including a variable bound by a previous segment of the pattern

- `unit` has to be a constant in range `1..=256`

- all parameters are optional
  - `size` and `unit` default to:
    - for `int`s, `unit=1, size=8`
    - for `float`s, `unit=1, size=64`; if set, `size` must be a literal `16`, `32` or `64`
    - for `bits`, `unit=1`; `size` expands to consume the whole unclaimed space
    - for `bin`s, `unit=8` and must be a multiple of `8`; `size` expands as above
    - for `str`, `unit=8` and cannot be set; `size` expands as above
    - for `char`s, neither can be set: the size depends on the character and its encoding
  - signedness is `unsigned` by default (can be `signed` or `unsigned`)
  - endianness is `big` by default (can be `big`, `little`, or `native`)
  - the encoding of a `char` is `utf8` by default (can be `utf8`, `utf16`, or `utf32`);
    endianness only applies to `utf16` and `utf32`
  - a string literal can be encoded the same way, e.g. `"łącko": str(utf16, little)`;
    other `str` values are always UTF-8

- the `size=` prefix is optional; `int(size=64)` is more commonly written as `int(64)`

- only the last segment can be unsized, and `unit` specifies its granularity, e.g.
  `rest: bits(unit=4)` matches a rest whose size is a multiple of 4 bits
  - most of the time, you'll match the rest with [`rest @ ..`](#the-rest) instead

- there are no alignment rules: segments of any size can follow each other

### Short form

If the only parameter you want to set is a fixed `size`, you can use a shortform syntax `x:16`,
with an integer literal. For matched variables, the type defaults to `int`.
When the size is provided by a variable, it needs the full form, e.g. `x: int(^len)`.

```rust,ignore,editable
// For matched variables, the type is assumed to be `int`.
// This line is equivalent to `<<x: int(16)>>`.
if let <<x:16>> = data {
    println!("data is a two-byte integer {}", x);
}

// The shorthand syntax comes in handy to match data with standard layout.
if let <<unix_ts_ms:48, ver:4, rand_a:12, var:2, rand_b:62>> = uuidv7_id {
    println!("{} {} {} {} {}", unix_ts_ms, ver, rand_a, var, rand_b);
}
```

If the type is omitted altogether, the size defaults to `8` as per the list above.
In that case `<<x>>` is equivalent to `<<x: int()>>` is equivalent to `<<x: int(8)>>`.

### The rest

The last segment of the matched binary can be `rest @ ..` (which binds whatever is left
of the value), or just `..` (which ignores it). The rest needs no type annotation;
it gets the narrowest type it is guaranteed to have:

- a `str`, when matching a `str`, and every segment before it is valid UTF-8
- a `bin`, when matching a `str` or a `bin` and the segments before it add up to whole bytes
- a `bits` otherwise

An explicit type can still be given. On a `bits`, `rest: bin()` acts as an additional check
and only matches when the `rest` is a whole number of bytes.

### Example

```rust,ignore,editable
pub fn main() {
    let bin = <<16, 5, 6, 7>>;

    match bin {
        // Bind `len`, then use it as the size of `payload`
        <<kind:8, len: int(8), payload: bin(^len), rest @ ..>> =>
            println!("kind = {}, payload = {:?}, rest = {:?}", kind, payload, rest),
        
        // The size can also be a guard expression
        <<kind:8, len: int(8), payload: bin(len / 2), rest @ ..>> =>
            println!("kind = {}, payload = {:?}, rest = {:?}", kind, payload, rest),

        _ => println!("invalid packet"),
    }

    let packet: bits = <<3:4, 5:4, 7:3>>;

    match packet {
        // The tail expands to match the whole rest of the binary.
        // Only the last segment can be unsized.
        // After an 8-bit header, the rest of a `bits` is a `bits`.
        <<header: int(8), tail @ ..>> =>
            println!("header = {}, tail = {:?}", header, tail),

        _ => println!("did not match"),
    }
}
```

## Literals

Literal segments can also be mixed directly into a binary pattern, whatever kind of value you are
matching. `str` and `b"…"` literals match their bytes, `char` literals are encoded as UTF-8 (unless
overridden), and integer and float literals are matched like `int` and `float` segments.

A literal that doesn't fit its segment, like `256:8`, can never match, and gets a compiler warning.

```rust,ignore,editable
pub fn main() {
    let text: str = "Patina!";
    let bytes: bin = <<b"PNG", '!', "rest">>;
    // For literals, types are optional unless you want to override a default.
    // Like with bindings, you set the type after a colon.
    let packet: bits = <<b"PT", '!', "ok", 5: int(3)>>;

    match text {
        // Every segment before the rest is valid UTF-8, so `rest` is a `str`.
        <<"Pa", 't', rest @ ..>> => println!("text rest {:?}", rest),
        _ => println!("unknown text"),
    }

    match bytes {
        <<b"PNG", '!', rest @ ..>> => println!("png data {:?}", rest),
        _ => println!("unknown binary"),
    }

    match packet {
        // The shorthand form can also be used with literals. 
        <<b"PT", '!', "ok", 5:3, rest @ ..>> =>
            println!("packet tail {:?}", rest),
        _ => println!("unknown packet"),
    }
}
```

## Bound variables

You can match against already bound variables' values using the pin operator `^`.

```rust,ignore,editable
pub fn main() {
    let png_prefix = "PNG";
    let jpg_prefix = "JPG";

    match "PNG!rest" {
        <<^png_prefix, rest @ ..>> => println!("png data {:?}", rest),
        <<^jpg_prefix, rest @ ..>> => println!("jpg data {:?}", rest),
        _ => println!("unknown data"),
    }

    let bit_prefix = 7;

    match <<119>> {
        // Like with literals, you can give the segment a type.
        <<^bit_prefix: int(16, little), rest @ ..>> => println!("expected {:?}", rest),
        // Or use the shorthand syntax.
        <<^bit_prefix:4, rest @ ..>> => println!("expected {:?}", rest),
        _ => println!("unexpected prefix"),
    }
}
```

## Building binaries

The same syntax as above can be used to build the binaries. Each segment is an expression, with
an optional type and spec. Without one, the type of the segment is determined by the type
of its value:

- `int`s become 8-bit integers
- `float`s become 64-bit floats
- `char`s and `str`s are encoded as UTF-8
- `bin` and `bits` are copied whole

Arithmetic in a segment needs parentheses, like `(len + 1):16`.

A built binary gets the narrowest type it is guaranteed to have:

- `str` if every segment is valid UTF-8 on its own
- `bin` if its size is a whole number of bytes
- `bits` otherwise.

```rust,ignore,editable
pub fn main() {
    let name = "Patina";
    let version = 3;

    // Every segment is valid UTF-8, so this is a `str`.
    let greeting: str = <<"Hello, ", name, '!'>>;

    // The size is a whole number of bytes, so this is a `bin`.
    let header: bin = <<b"PT", version:16, 1.5: float(32), 'é': char(utf16)>>;

    // Anything else is a `bits`.
    let flags: bits = <<1:1, 0:1, (version + 1): int(3)>>;

    println!("{:?} {:?} {:?}", greeting, header, flags);
}
```

### See also:

[`match`](../match.md), [Guards](guard.md), and [Primitives](../../primitives.md)

[erlang bit syntax]: https://www.erlang.org/doc/system/bit_syntax.html#introduction
[bound variables]: #bound-variables
