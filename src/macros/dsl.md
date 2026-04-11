# Domain Specific Languages (DSLs)

A DSL is a mini "language" embedded in a Patina macro. It is completely valid
Patina because the macro system expands into normal Patina constructs, but it looks
like a small language. This allows you to define concise or intuitive syntax for
some special functionality (within bounds).

Suppose that I want to define a little calculator API. I would like to supply
an expression and have the output printed to console.

```rust,ignore,editable
macro_rules! calculate {
    (eval $e:expr) => {
        {
            let val: int = $e; // Force types to integers
            println!("{} = {}", stringify!{$e}, val);
        }
    };
}

pub fn main() {
    calculate! {
        eval 1 + 2 // hehehe `eval` is _not_ a Patina keyword!
    }

    calculate! {
        eval (1 + 2) * (3 / 4)
    }
}
```

Output:

```txt
1 + 2 = 3
(1 + 2) * (3 / 4) = 0
```

Note the two pairs of braces in the macro. The outer ones are
part of the syntax of `macro_rules!`, in addition to `()` or `[]`.
