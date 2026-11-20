# Pulling `Result`s out of `Option`s

The most basic way of handling mixed error types is to just embed them in each
other.

```rust,ignore,editable
use std::num::ParseIntError;

fn double_first(list: [str]) -> Option<Result<int, ParseIntError>> {
    list.first().map(|first| {
        first.parse::<int>().map(|n| 2 * n)
    })
}

pub fn main() {
    let numbers = ["42", "93", "18"];
    let empty = [];
    let strings = ["tofu", "93", "18"];

    println!("The first doubled is {:?}", double_first(numbers));

    println!("The first doubled is {:?}", double_first(empty));
    // Error 1: the input list is empty

    println!("The first doubled is {:?}", double_first(strings));
    // Error 2: the element doesn't parse to a number
}
```

There are times when we'll want to stop processing on errors (like with
[`?`][enter_question_mark]) but keep going when the `Option` is `None`. The `transpose` function comes in handy to swap the `Result` and `Option`.

```rust,ignore,editable
use std::num::ParseIntError;

fn double_first(list: [str]) -> Result<Option<int>, ParseIntError> {
    let opt = list.first().map(|first| {
        first.parse::<int>().map(|n| 2 * n)
    });

    opt.transpose()
}

pub fn main() {
    let numbers = ["42", "93", "18"];
    let empty = [];
    let strings = ["tofu", "93", "18"];

    println!("The first doubled is {:?}", double_first(numbers));
    println!("The first doubled is {:?}", double_first(empty));
    println!("The first doubled is {:?}", double_first(strings));
}
```

[enter_question_mark]: ../result/enter_question_mark.md
