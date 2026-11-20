# Iterators

The `Iterator` trait is used to implement iterators over collections
such as lists.

The trait requires only a method to be defined for the `next` element,
which may be manually defined in an `impl` block or automatically
defined (as in lists and ranges).

As a point of convenience for common situations, the `for` construct
turns some collections into iterators using the `.iter()` method.

```rust,ignore,editable
struct Fibonacci {
    curr: int,
    next: int,
}

// Implement `Iterator` for `Fibonacci`.
// The `Iterator` trait only requires a method to be defined for the `next` element,
// and an `associated type` to declare the return type of the iterator.
impl Iterator for Fibonacci {
    // We can refer to this type using Self::Item
    type Item = int;

    // Here, we define the sequence using `.curr` and `.next`.
    // The return type is `Option<T>`:
    //     * When the `Iterator` is finished, `None` is returned.
    //     * Otherwise, the next value is wrapped in `Some` and returned.
    // We use Self::Item in the return type, so we can change
    // the type without having to update the function signatures.
    fn next(self) -> (Self, Option<Self::Item>) {
        let next_self = Fibonacci { curr: self.next, next: self.curr + self.next }

        // Since there's no endpoint to a Fibonacci sequence, the `Iterator`
        // will never return `None`, and `Some` is always returned.
        (next_self, Some(self.curr))
    }
}

// Returns a Fibonacci sequence generator
fn fibonacci() -> Fibonacci {
    Fibonacci { curr: 0, next: 1 }
}

pub fn main() {
    // `0..3` is an `Iterator` that generates: 0, 1, and 2.
    let mut sequence = 0..3;
    let mut val: Option<int> = None;

    println!("Four consecutive `next` calls on 0..3");
    (sequence, val) = sequence.next();
    println!("> {:?}", val);
    (sequence, val) = sequence.next();
    println!("> {:?}", val);
    (sequence, val) = sequence.next();
    println!("> {:?}", val);
    (sequence, val) = sequence.next();
    println!("> {:?}", val);

    // `for` works through an `Iterator` until it returns `None`.
    // Each `Some` value is unwrapped and bound to a variable (here, `i`).
    println!("Iterate through 0..3 using `for`");
    for i in 0..3 {
        println!("> {}", i);
    }

    // The `take(n)` method reduces an `Iterator` to its first `n` terms.
    println!("The first four terms of the Fibonacci sequence are: ");
    for i in fibonacci().take(4) {
        println!("> {}", i);
    }

    // The `skip(n)` method shortens an `Iterator` by dropping its first `n` terms.
    println!("The next four terms of the Fibonacci sequence are: ");
    for i in fibonacci().skip(4).take(4) {
        println!("> {}", i);
    }

    let list = [1, 3, 3, 7];

    // The `iter` method produces an `Iterator` over a list
    println!("Iterate the following list {:?}", list);
    for i in list.iter() {
        println!("> {}", i);
    }
    // For implicitly calls `iter()` if the supplied value is not an iterator
    println!("Iterate the following list {:?} with implicit .iter()", list);
    for i in list {
        println!("> {}", i);
    }
}
```
