# Flow of Control

An integral part of any programming language are ways to modify control flow:
`if`/`else`, `for`, and others. Let's talk about them in Patina.

### Variable rebinding inside flow control

Unlike many other languages on BEAM, in most control flow blocks Patina code can rebind variables
in a way that is visible outside of the block.

```rust,ignore,editable
pub fn main() {
    // only `mut` bindings can be rebound
    let mut n = 5;

    if n > 4 {
        n = 4;
    }

    println!("{:?}", n);
    // 4
}
```

This also works inside loops! However, there are two notable exceptions: function closures and `try`
blocks. Inside these, a `let mut` variable behaves as an immutable `let`.

```rust,ignore,editable
pub fn main() {
    let mut n = 1;

    while n < 10 {
        n += n;
    }

    println!("{:?}", n);
    // 16

    let fun = ||{
        n = 20;
        // ^ERROR! External variable `n` cannot be rebound inside a closure
    };

    try {
        n = 30;
        // ^ERROR! External variable `n` cannot be rebound inside a try block
    } catch {
        throw(1) => println!("caught 1!"),
    }
}
```
