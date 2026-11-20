# File hierarchy

Modules can be mapped to a file/directory hierarchy. Let's break down the
[visibility example][visibility] in files:

```shell
$ tree .
.
├── my_mod
│   └── inaccessible.pt
│   └── nested.pt
├── my_mod.pt
└── split.pt
```

In `split.pt`:

```rust,ignore
// This declaration will look for a file named `my_mod.pt` and will
// insert its contents inside a module named `my_mod` under this scope
mod my_mod;

fn function() {
    println!("called `function()`");
}

pub fn main() {
    my_mod::function();

    function();

    my_mod::indirect_access();

    my_mod::nested::function();
}

```

In `my_mod.pt`:

```rust,ignore
// Similarly `mod inaccessible` and `mod nested` will locate the `nested.pt`
// and `inaccessible.pt` files and insert them here under their respective
// modules
mod inaccessible;
pub mod nested;

pub fn function() {
    println!("called `my_mod::function()`");
}

fn private_function() {
    println!("called `my_mod::private_function()`");
}

pub fn indirect_access() {
    print!("called `my_mod::indirect_access()`, that\n> ");

    private_function();
}
```

In `my_mod/nested.pt`:

```rust,ignore
pub fn function() {
    println!("called `my_mod::nested::function()`");
}

#[allow(dead_code)]
fn private_function() {
    println!("called `my_mod::nested::private_function()`");
}
```

In `my_mod/inaccessible.pt`:

```rust,ignore
#[allow(dead_code)]
pub fn public_function() {
    println!("called `my_mod::inaccessible::public_function()`");
}
```

Let's check that things still work as before:

```shell
$ patinac split.pt && patina run -m Split
called `my_mod::function()`
called `function()`
called `my_mod::indirect_access()`, that
> called `my_mod::private_function()`
called `my_mod::nested::function()`
```

[visibility]: visibility.md
