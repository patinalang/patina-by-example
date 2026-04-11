# Associated types

The use of "Associated types" improves the overall readability of code
by moving inner types locally into a trait as *output* types. Syntax
for the `trait` definition is as follows:

```rust,ignore
// `A` and `B` are defined in the trait via the `type` keyword.
// (Note: `type` in this context is different from `type` when used for
// aliases).
trait Contains {
    type A;
    type B;

    // Updated syntax to refer to these new types generically.
    fn contains(self, _: Self::A, _: Self::B) -> bool;
}
```

Note that functions that use the `trait` `Contains` are no longer required
to express `A` or `B` at all:

```rust,ignore
// Without using associated types
fn difference<A, B, C>(container: C) -> int where
    C: Contains<A, B> { ... }

// Using associated types
fn difference<C: Contains>(container: C) -> int { ... }
```

Let's rewrite the example from the previous section using associated types:

```rust,ignore,editable
struct Container(int, int);

// A trait which checks if 2 items are stored inside of container.
// Also retrieves first or last value.
trait Contains {
    // Define generic types here which methods will be able to utilize.
    type A;
    type B;

    fn contains(self, _: Self::A, _: Self::B) -> bool;
    fn first(self) -> int;
    fn last(self) -> int;
}

impl Contains for Container {
    // Specify what types `A` and `B` are. If the `input` type
    // is `Container(int, int)`, the `output` types are determined
    // as `int` and `int`.
    type A = int;
    type B = int;

    // `Self::A` and `Self::B` are also valid here.
    fn contains(self, number_1: int, number_2: int) -> bool {
        (self.0 == number_1) && (self.1 == number_2)
    }
    // Grab the first number.
    fn first(self) -> int { self.0 }

    // Grab the last number.
    fn last(self) -> int { self.1 }
}

fn difference<C: Contains>(container: C) -> int {
    container.last() - container.first()
}

pub fn main() {
    let number_1 = 3;
    let number_2 = 10;

    let container = Container(number_1, number_2);

    println!("Does container contain {} and {}: {}",
        number_1, number_2,
        container.contains(number_1, number_2));
    println!("First number: {}", container.first());
    println!("Last number: {}", container.last());

    println!("The difference is: {}", difference(container));
}
```
