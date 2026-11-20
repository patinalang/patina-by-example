# Unpacking options and defaults

There is more than one way to unpack an `Option` and fall back on a default if it is `None`. To choose the one that meets our needs, we need to consider whether we want eager or lazy evaluation.

## `or()` is chainable, evaluates eagerly

`or()`is chainable and eagerly evaluates its argument, as is shown in the following example. Note that `or`'s arguments are evaluated even when they end up unused.

```rust,ignore,editable
#[derive(Debug)]
enum Fruit { Apple, Orange, Banana, Kiwi, Lemon }

pub fn main() {
    let apple = Some(Fruit::Apple);
    let orange = Some(Fruit::Orange);
    let no_fruit: Option<Fruit> = None;

    let first_available_fruit = no_fruit.or(orange).or(apple);
    println!("first_available_fruit: {:?}", first_available_fruit);
    // first_available_fruit: Some(Orange)

    // In the example above, `or(orange)` returned a `Some`, so the value of `apple` was not needed.
    // Values are never moved in Patina, so `apple` can still be used afterwards.
    println!("apple is still available: {:?}", apple);
    // apple is still available: Some(Apple)
}
```

## `or_else()` is chainable, evaluates lazily

Another alternative is to use `or_else`, which is also chainable, and evaluates lazily, as is shown in the following example:

```rust,ignore,editable
#[derive(Debug)]
enum Fruit { Apple, Orange, Banana, Kiwi, Lemon }

pub fn main() {
    let no_fruit: Option<Fruit> = None;
    let get_kiwi_as_fallback = || {
        println!("Providing kiwi as fallback");
        Some(Fruit::Kiwi)
    };
    let get_lemon_as_fallback = || {
        println!("Providing lemon as fallback");
        Some(Fruit::Lemon)
    };

    let first_available_fruit = no_fruit
        .or_else(get_kiwi_as_fallback)
        .or_else(get_lemon_as_fallback);
    println!("first_available_fruit: {:?}", first_available_fruit);
    // Providing kiwi as fallback
    // first_available_fruit: Some(Kiwi)
}
```
