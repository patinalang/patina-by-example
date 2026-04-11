# Returning Traits with `dyn`

Patina tries to represent `struct`s and `enums` efficiently. For example, single-field structs
are by default represented in runtime as that field alone, removing the overhead associated with
(un)wrapping values. `struct Celcius(int)` will just be an integer in the BEAM VM runtime, and
`let c = Celcius(10); c.is_freezing` will statically dispatch the call to the `Celcius` module.

This is why, similarly to Rust, all your functions have to return a concrete type. Just like Rust,
if you have a trait like `Animal`, you can't write a function that returns `Animal`, because Patina
may not have enough runtime information to dispatch function calls to the concrete implementation.

However, you can *make* Patina store enough information by using the `dyn` keyword. Objects returned
as `dyn Animal` will be represented with additional runtime information, and calling `Animal`
methods will require an additional dynamic dispatch - this is why `dyn` is clearly marked in code,
instead of being an implicit behaviour.

```rust,ignore,editable
struct Sheep {}
struct Cow {}

trait Animal {
    // Instance method signature
    fn noise(self) -> str;
}

// Implement the `Animal` trait for `Sheep`.
impl Animal for Sheep {
    fn noise(self) -> str {
        "baaaaah!"
    }
}

// Implement the `Animal` trait for `Cow`.
impl Animal for Cow {
    fn noise(self) -> str {
        "moooooo!"
    }
}

// Returns some struct that implements Animal, but we don't know which one at compile time.
fn random_animal(random_number: float) -> dyn Animal {
    if random_number < 0.5 {
        Sheep {}
    } else {
        Cow {}
    }
}

pub fn main() {
    let random_number = 0.234;
    let animal = random_animal(random_number);
    println!("You've randomly chosen an animal, and it says {}", animal.noise());
}
```
