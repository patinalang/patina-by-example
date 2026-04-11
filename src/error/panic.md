# `panic`

The simplest error handling mechanism we will see is `panic`. It prints an
error message, starts unwinding the stack, and usually exits the BEAM process.
Here, we explicitly call `panic` on our error condition:

```rust,editable,ignore
fn drink(beverage: str) {
    // You shouldn't drink too many sugary beverages.
    if beverage == "lemonade" { panic!("AAAaaaaa!!!!"); }

    println!("Some refreshing {} is all I need.", beverage);
}

pub fn main() {
    drink("water");
    drink("lemonade");
    drink("still water");
}
```

The first call to `drink` works. The second panics and thus the third is never called.

Patina's `panic` is a BEAM error.

## See also:

[try/catch/after][try catch after]

[try catch after]: ../flow_control/try_catch_after.md
