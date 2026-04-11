# Hello World

This is the source code of the traditional Hello World program.

```rust,ignore,editable
// hello.pt

// This is a comment, and is ignored by the compiler.
// You can test this code by clicking the "Run" button over there ->
// or if you prefer to use your keyboard, you can use the "Ctrl + Enter"
// shortcut.

// This code is editable, feel free to hack it!
// You can always return to the original code by clicking the "Reset" button ->

// This is the main function.
pub fn main() {
    // Statements here are executed when the compiled binary is called.

    // Print text to the console.
    println!("Hello World!");
}
```

`println!` is a [*macro*][macros] that prints text to the
console.

A BEAM module can be generated using the Patina compiler: `patinac`.

```bash
$ patinac hello.pt
```

`patinac` will produce a `.beam` file that can be executed with `patina run`.

```bash
$ patina run -m Hello
Hello World!
```

[macros]: macros.md
