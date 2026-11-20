# Traits

Of course `trait`s can also be generic. Here we define a `SeededHash` trait that hashes
itself and an input.

```rust,ignore,editable
extern mod erlang;

struct Empty;
struct StaticSeed;

// A trait generic over `T`.
trait SeededHash<T> {
    // Define a method on the caller type which takes an
    // additional single parameter `T`.
    fn seeded_hash(self, seed: T) -> int;
}

// Implement `SeededHash<T>` for any generic parameter `T` and
// caller `U`.
impl<T, U> SeededHash<T> for U {
    fn seeded_hash(self, seed: T) -> int {
        unsafe { erlang::phash2((seed, self)).cast_unchecked::<int>() }
    }
}

pub fn main() {
    let empty = Empty;
    let seed  = StaticSeed;

    // hash `empty` with static seed
    empty.seeded_hash(seed);
}
```

### See also:

[`struct`][structs], and [`trait`][traits]

[structs]: ../custom_types/structs.md
[traits]: ../trait.md
