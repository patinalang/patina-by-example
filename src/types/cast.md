# Casting

Patina provides no implicit type conversion (coercion) between primitive types.
But, explicit type conversion (casting) can be performed using the `as` keyword.

```rust,editable,ignore
pub fn main() {
    let decimal = 65.4321;

    // Error! No implicit conversion
    let integer: int = decimal;

    // Explicit conversion
    let integer = decimal as int; // casting float to integer is equivalent to decimal.trunc()
    let character = integer as char;

    // Error! There are limitations in conversion rules.
    // A float cannot be directly converted to a char.
    let character = decimal as char;
    // FIXME ^ Comment out this line

    println!("Casting: {} -> {} -> {}", decimal, integer, character);

    // str can be cast to bin or bits
    let string = "hello";
    let binary = string as bin;
    let bitstring = string as bits;

    // bin can be cast to bits
    let bitstring = binary as bits;

    // PATINA TODO: how useful is the byte type, really?
    // integer literal can be cast to a byte
    // let byte_val = 123 as byte;

    // byte can be cast to an integer, but not the other way around!
    let integer_from_byte = byte_val as int;
    // Error! Byte cannot be cast from an arbitrary integer
    let byte_from_integer = integer_from_byte as byte;
}
```
