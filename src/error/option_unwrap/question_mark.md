# Unpacking options with `?`

You can unpack `Option`s by using `match` statements, but it's often easier to
use the `?` operator. If `x` is an `Option`, then evaluating `x?` will return
the underlying value if `x` is `Some`, otherwise it will terminate whatever
function is being executed and return `None`.

```rust,ignore
fn next_birthday(current_age: Option<int>) -> Option<str> {
    // If `current_age` is `None`, this returns `None`.
    // If `current_age` is `Some`, the inner `int` value + 1
    // gets assigned to `next_age`
    let next_age: int = current_age? + 1;
    Some(format!("Next year I will be {}", next_age))
}
```

You can chain many `?`s together to make your code much more readable.

```rust,ignore,editable
struct Person {
    job: Option<Job>,
}

struct Job {
    phone_number: Option<PhoneNumber>,
}

#[allow(dead_code)]
struct PhoneNumber {
    area_code: Option<int>,
    number: int,
}

impl Person {
    // Gets the area code of the phone number of the person's job, if it exists.
    fn work_phone_area_code(self) -> Option<int> {
        // This would need many nested `match` statements without the `?` operator.
        // It would take a lot more code - try writing it yourself and see which
        // is easier.
        self.job?.phone_number?.area_code
    }
}

pub fn main() {
    let p = Person {
        job: Some(Job {
            phone_number: Some(PhoneNumber {
                area_code: Some(61),
                number: 439222222,
            }),
        }),
    };

    assert_eq!(p.work_phone_area_code(), Some(61));
}
```
