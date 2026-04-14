# Runtime representation

Patina has multiple possible runtime representations of its structs. The default representation,
`#[repr(Patina)]`, is equivalent to:

- `#[repr(atom)]` for unit structs like `struct Foo` or `struct Bar()`,
- `#[repr(transparent)]` for single-element structs like `struct Foo(int)` or `struct Bar(x: int)`,
- `#[repr(Elixir)]` for Elixir-style structs like `struct Foo{x: int, y: float}`,
- `#[repr(tagged)]` for tuple structs like `struct Foo(int, float)`.

Regardless of struct definition style, each of these representations can be forced through a `repr`
attribute. There are caveats: `repr(atom)` can only be set for unit structs, and `repr(transparent)`
for single-element structs.

If you need a particular representation, it's a good idea to explicitly
opt-in via `repr(...)`, even if it's the default - Patina compiler will help you maintain
the invariant.

## Transparent structs

By default, single-element structs are represented in runtime as the element itself.

```rust,ignore,editable
// omitting repr is equivalent to `#[repr(Patina)]`
struct Celcius(degrees: float);

pub fn main() {
    println("{:?}", Celcius(2.345) as term);
    // 2.345
}
```

## Elixir-style structs

These are encoded as fully Elixir-compatible structs, with atom fields,
a `__struct__: module` field, and a module that implements `__struct__/0`,
`__struct__/1` and `__info__(:struct)` callbacks.

```rust,ignore,editable
// repr(Elixir) is the default representation for Elixir-style structs
// with more than one field name.
struct Point {
    x: float,
    y: float,
}

// It can be set explicitly for other struct definition styles.
#[repr(Elixir)]
struct Foo(int)

pub fn main() {
    println("{:?}", Point{x: 1, y: 2} as term);
    // #{a"__struct__" => a"Elixir.Patina.ByExample.Point", a"x" => 5.2, a"y" => 0.4}

    println("{:?}", Foo(42) as term);
    // #{a"__struct__" => a"Elixir.Patina.ByExample.Foo", a"0" => 42}
}
```

## Tuple structs

Encoded as tagged tuples, where the first element is the atom uniquely identifying
the struct name, and the following tuple fields are the struct fields. This is similar
to Erlang's record encoding.

```rust,ignore,editable
// repr(tagged) is the default represenation for tuple structs
// with more than one field
struct Pair(int, float);

// It can be set explicitly for other struct definitino styles.
#[repr(tagged)]
struct Bar{ x: int, y: int }

pub fn main() {
    println("{:?}", Pair(42, 3.14) as term);
    // (a"Elixir.Patina.ByExample.Pair", 42, 3.14)

    println("{:?}", Bar(x: 1, y: 2) as term);
    // (a"Elixir.Patina.ByExample.Bar", 1, 2)
}
```

## Unit structs

Unit structs are encoded directly as their atom names.

```rust,ignore,editable
struct Unit;

pub fn main() {
    println("{:?}", Unit as term);
    // a"Elixir.Patina.ByExample.Unit"
}
```
