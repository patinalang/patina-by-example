# Binary matching

A `match` can destructure `str`ings, `bin`aries, and `bits`trings directly.

Like in [Erlang's bit syntax][erlang bit syntax], binaries are matched in *segments*.
A binary pattern only matches if every segment matches, and the whole value is consumed.

```rust,ignore,editable
pub fn main() {
    let payload = <<1, 17, 42, 99>>;

    match payload {
        <<prefix @ int(16), tag @ int(8), rest @ bin()>> =>
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
`var @ int(size=x, signed, little, unit=8)`, where:

- segments can be bound to variables with `@`

- `size` can be 0 or more, and can be given through a variable - including a variable bound
  by a previous segment of the pattern

- `unit` has to be a constant in range `1..=256`

- all parameters are optional
  - `size` and `unit` default to:
    - for `int`s, `unit=1, size=8` 
    - for `float`s, `unit=1, size=64`; if set, `size` must be a literal `32` or `64`
    - for `bits`, `unit=1`; `size` expands to consume the whole unclaimed space 
    - for `bin`s, `unit=8` and must be a multiple of `8`; `size` expands as above
    - for `str`, `unit=8` and cannot be set; `size` expands as above
  - signedness is `unsigned` by default (can be `signed` or `unsigned`)
  - endianness is `big` by default (can be `big`, `little`, or `native`)

- the `size=` prefix is optional; `int(size=64)` is more commonly written as `int(64)`

- there are no alignment rules for individual `int` and `float` segments
  - for `bin` and `bits` without size, `unit` specifies the alignment.

### Short form

If the only parameter you want to set is `size`, you can use a shortform syntax `x:size`.
For matched variables, the type defaults to `int`.

```rust,ignore,editable
// For matched variables, the type is assumed to be `int`.
// This line is equivalent to `<<x @ int(16)>>`.
if let <<x:16>> = data {
    println!("data is a two-byte integer {}", x);
}

// The shorthand syntax comes in handy to match data with standard layout.
if let <<unix_ts_ms:48, ver:4, rand_a:12, var:2, rand_b:62>> = uuidv7_id {
    println!("{} {} {} {} {}", unix_ts_ms, ver, rand_a, var, rand_b);
}
```

If the `:size` suffix is omitted, it defaults to `8` as per the list above.
In that case `<<x>>` is equivalent to `<<x @ int()>>.

### Example

```rust,ignore,editable
pub fn main() {
    let bin = <<16, 5, 6, 7>>;

    match bin {
        // Bind `len`, then use it as the size of `payload`
        <<kind:8, len @ int(8), payload @ bin(len), rest @ bin()>> =>
            println!("kind = {}, payload = {:?}, rest = {:?}", kind, payload, rest),
        
        // The size can also be a guard expression
        <<kind:8, len @ int(8), payload @ bin(len / 2), rest @ bin()>> =>
            println!("kind = {}, payload = {:?}, rest = {:?}", kind, payload, rest),

        _ => println!("invalid packet"),
    }

    let packet: bits = <<3:4, 5:4, 7:3>>;

    match packet {
        // The tail expands to match the whole rest of the binary.
        <<header @ int(8), tail @ bits()>> =>
            println!("header = {}, tail = {:?}", header, tail),

        // The unsized segment can be in the middle, as long as both the prefix
        // and suffix sizes are known at compilation time.
        <<header, mid @ bits(), tail @ float()>> =>
            println!("header = {}, mid = {:?}, tail = {:?}", header, mid tail),

        _ => println!("did not match"),
    }
}
```

## Literals

Literal segments can also be mixed directly into a binary pattern. The allowed literal types depend
on what kind of value you are matching:

- matching `bits`: `str`, `char`, `bin`, `bits`, `int`
- matching `bin`: `str`, `char`, `bin`
  - `int` and `bits` are allowed if their size is constant and divisible by 8,
  or their `unit`s are all divisible by 8
- matching `str`: `str`, `char`

```rust,ignore,editable
pub fn main() {
    let text: str = "Patina!";
    let bytes: bin = <<b"PNG", '!', "rest">>;
    // For literals, types are optional unless you want to override a default.
    // In that case, you use `as type` instead of `@ type()`.
    let packet: bits = <<b"PT", '!', "ok", 5 as int(3)>>;

    match text {
        <<"Pa", 't', rest @ str()>> => println!("text rest {:?}", rest),
        _ => println!("unknown text"),
    }

    match bytes {
        <<b"PNG", '!', rest @ bin()>> => println!("png data {:?}", rest),
        _ => println!("unknown binary"),
    }

    match packet {
        // The shorthand form can also be used with literals. 
        <<b"PT", '!', "ok", 5:3, rest: bits()>> =>
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
        <<^png_prefix, rest @ str()>> => println!("png data {:?}", rest),
        <<^jpg_prefix, rest @ str()>> => println!("jpg data {:?}", rest),
        _ => println!("unknown data"),
    }

    let bit_prefix = 7;

    match <<119>> {
        // Like with literals, you can use `as` to specify parameters.
        <<^bit_prefix as int(16, little), rest @ bits()>> => println!("expected {:?}", rest),
        // Or use the shorthand syntax.
        <<^bit_prefix:4, rest @ bits()>> => println!("expected {:?}", rest),
        _ => println!("unexpected prefix"),
    }
}
```

### See also:

[`match`](../match.md), [Guards](guard.md), and [Primitives](../../primitives.md)

[erlang bit syntax]: https://www.erlang.org/doc/system/bit_syntax.html#introduction
