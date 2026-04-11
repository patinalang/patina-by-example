# Primitives

Patina provides access to a variety of `primitives`, most of them mapping directly to [BEAM
primitive types][BEAM data types]. A sample includes:

### Scalar Types

* `atom` Literal constants with name
* `int` Signed variable-size integers (up to 2^(2^31))
* `float` Floating point values (64-bit)
* `char` Unicode scalar values like `'a'`, `'α'` and `'∞'`
* `bool` either `true` or `false`
<!-- Patina TODO * `byte` Single byte values in 0-255 range -->
* `bits` [Bit String][bitstring] - a contiguous store of a number of bits not
  necessarily divisible by 8
* `bin` Binary; a subtype of `bits` with length divisible by 8
* `str` UTF-8 encoded string; a subtype of `bin`
* `fnref()` Stateless function reference, similar to Elixir's `&Foo.bar/1`
* `fn()` Callable function, may be a stateless reference or a closure
* The unit type `()`, whose only possible value is an empty tuple: `()`

Despite the value of a unit type being a tuple, it is not considered a compound
type because it does not contain multiple values.

### Compound Types

* Lists like `[1, 2, 3]`
* Tuples like `(1, true)`
* Maps like `#{"foo" => 1, "bar" => 2}`

Variables can always be *type annotated*.
Note that Patina can also infer types from context.

```rust,editable,ignore
pub fn main() {
    // Variables can be type annotated.
    let logical: bool  = true;
    let a_float: float = 1.0;

    // A type can also be inferred from context.
    let inferred_type = 12; // Type float is inferred from another line.
    inferred_type = add_floats(inferred_type, 3.14)

    // A mutable variable's value can be changed - rebound to a new value.
    let mut mutable = 12; // Mutable `int`
    mutable = 21;

    // Error! The type of a variable can't be changed.
    mutable = true;

    // Variables can be overwritten with shadowing.
    let mutable = true;

    /* Compound types - List, Tuple and Maps */

    // List's signature consists of type T as [T].
    let my_list: [int] = [1, 2, 3, 4, 5];

    // Tuple is a collection of values of different types
    // and is constructed using parentheses ().
    let my_tuple = (5, true, -5.04);

    // Map's signature consists of Key type K and Value type V as #{K => V}
    let my_map: #{float => str} = #{3.14 => "pi", 0 => "zero"}

    /* Function types */

    let add_ref: fnref(float, float) -> float = add_float;
    // fnref is implicitly convertible to fn
    let add_fn: fn(float, float) -> float = add_ref;
}

fn add_floats(a: float, b: float) -> float {
    a + b
}
```

### Runtime representation

* `bits`, `bin` and `str` are all the same Erlang binary data type.
  While some values of these types are distinguishable (an Erlang binary that
  doesn't contain valid UTF-8 data is definitely not a `str`), in general you
  can't rely on runtime checks to differentiate a `bin` from a `str` or `bits`.

* `char` values are represented by single integers
<!-- Patina TODO * `byte` values are represented by single integers in the 0-255 range -->
* `bool` values are represented as atoms `a"true"` and `a"false"`
* all of the other primitive types are directly represented in BEAM

### See also:

[`mut`][mut], [`inference`][inference], and [`shadowing`][shadowing]

[mut]: variable_bindings/mut.md
[inference]: types/inference.md
[shadowing]: variable_bindings/scope.md
[BEAM data types]: https://www.erlang.org/doc/system/data_types.html
[bitstring]: https://www.erlang.org/doc/system/data_types.html#bit-strings-and-binaries
