# Iterating over `Result`s

An `Iter::map` operation might fail, for example:

```rust,ignore,editable
pub fn main() {
    let strings = ["tofu", "93", "18"];
    let numbers: [_] = strings
        .into_iter()
        .map(|s| s.parse::<int>())
        .collect();
    println!("Results: {:?}", numbers);
}
```

Let's step through strategies for handling this.

## Ignore the failed items with `filter_map()`

`filter_map` calls a function and filters out the results that are `None`.

```rust,ignore,editable
pub fn main() {
    let strings = ["tofu", "93", "18"];
    let numbers: [_] = strings
        .into_iter()
        .filter_map(|s| s.parse::<int>().ok())
        .collect();
    println!("Results: {:?}", numbers);
}
```

## Fail the entire operation with `collect()`

`Result` implements `FromIterator` so that a list of results (`[Result<T, E>]`)
can be turned into a result with a list (`Result<[T], E>`). Once an
`Result::Err` is found, the iteration will terminate.

```rust,ignore,editable
pub fn main() {
    let strings = ["tofu", "93", "18"];
    let numbers: Result<[_], _> = strings
        .into_iter()
        .map(|s| s.parse::<int>())
        .collect();
    println!("Results: {:?}", numbers);
}
```

This same technique can be used with `Option`.

## Collect all valid values and failures with `partition()`

```rust,ignore,editable
pub fn main() {
    let strings = ["tofu", "93", "18"];
    let (numbers, errors): ([_], [_]) = strings
        .into_iter()
        .map(|s| s.parse::<int>())
        .partition(Result::is_ok);
    println!("Numbers: {:?}", numbers);
    println!("Errors: {:?}", errors);
}
```

When you look at the results, you'll note that everything is still wrapped in
`Result`.  A little more boilerplate is needed for this.

```rust,ignore,editable
pub fn main() {
    let strings = ["tofu", "93", "18"];
    let (numbers, errors): ([_], [_]) = strings
        .into_iter()
        .map(|s| s.parse::<int>())
        .partition(Result::is_ok);
    let numbers: [_] = numbers.into_iter().map(Result::unwrap).collect();
    let errors: [_] = errors.into_iter().map(Result::unwrap_err).collect();
    println!("Numbers: {:?}", numbers);
    println!("Errors: {:?}", errors);
}
```
