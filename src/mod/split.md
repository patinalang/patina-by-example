# File hierarchy

Modules can be mapped to a file/directory hierarchy. Let's break down the
[visibility example][visibility] in files:

```shell
$ tree .
.
├── my_mod
│   └── inaccessible.rs
│   └── nested.rs
├── my_mod.rs
└── split.rs
```

In `split.rs`:

```rust,ignore
// This declaration will look for a file named `my_mod.rs` and will
// insert its contents inside a module named `MyMod` under this scope
mod MyMod;

fn function() {
    println!("called `function()`");
}

pub fn main() {
    MyMod::function();

    function();

    MyMod::indirect_access();

    MyMod::Nested::function();
}

```

In `my_mod.rs`:

```rust,ignore
// Similarly `mod inaccessible` and `mod nested` will locate the `nested.rs`
// and `inaccessible.rs` files and insert them here under their respective
// modules
mod Inaccessible;
pub mod Nested;

pub fn function() {
    println!("called `MyMod::function()`");
}

fn private_function() {
    println!("called `MyMod::private_function()`");
}

pub fn indirect_access() {
    print!("called `MyMod::indirect_access()`, that\n> ");

    private_function();
}
```

In `my_mod/nested.rs`:

```rust,ignore
pub fn function() {
    println!("called `MyMod::Nested::function()`");
}

#[allow(dead_code)]
fn private_function() {
    println!("called `MyMod::Nested::private_function()`");
}
```

In `my_mod/inaccessible.rs`:

```rust,ignore
#[allow(dead_code)]
pub fn public_function() {
    println!("called `MyMod::Inaccessible::public_function()`");
}
```

Let's check that things still work as before:

```shell
$ patinac split.pt && patina run -m Split
called `MyMod::function()`
called `function()`
called `MyMod::indirect_access()`, that
> called `MyMod::private_function()`
called `MyMod::Nested::function()`
```

[visibility]: visibility.md
