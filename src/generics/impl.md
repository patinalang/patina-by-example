# Implementation

Similar to functions, implementations require care to remain generic.

```rust,ignore
struct S; // Concrete type `S`
struct GenericVal<T>(T); // Generic type `GenericVal`

// impl of GenericVal where we explicitly specify type parameters:
impl GenericVal<float> {} // Specify `float`
impl GenericVal<S> {} // Specify `S` as defined above

// `<T>` Must precede the type to remain generic
impl<T> GenericVal<T> {}
```

```rust,ignore,editable
struct Val {
    val: int,
}

struct GenVal<T> {
    gen_val: T,
}

// impl of Val
impl Val {
    fn value(self) -> int {
        self.val
    }
}

// impl of GenVal for a generic type `T`
impl<T> GenVal<T> {
    fn value(self) -> T {
        self.gen_val
    }
}

pub fn main() {
    let x = Val { val: 3 };
    let y = GenVal { gen_val: 42 };

    println!("{}, {}", x.value(), y.value());
}
```

### See also:

[`impl`][methods], and [`struct`][structs]

[methods]: ../fn/methods.md
[structs]: ../custom_types/structs.md
