# `TryFrom` and `TryInto`

Similar to [`From` and `Into`][from-into], [`TryFrom`] and [`TryInto`] are
generic traits for converting between types. Unlike `From`/`Into`, the
`TryFrom`/`TryInto` traits are used for fallible conversions, and as such,
return [`Result`]s.

[from-into]: from_into.html
[`TryFrom`]: try_from_try_into.md
[`TryInto`]: try_from_try_into.md
[`Result`]: ../error/result.md

```rust,ignore,editable
use std::convert::TryFrom;
use std::convert::TryInto;

#[derive(Debug, PartialEq)]
struct EvenNumber(int);

impl TryFrom<int> for EvenNumber {
    type Error = ();

    fn try_from(value: int) -> Result<Self, Self::Error> {
        if value % 2 == 0 {
            Ok(EvenNumber(value))
        } else {
            Err(())
        }
    }
}

pub fn main() {
    // TryFrom

    assert_eq!(EvenNumber::try_from(8), Ok(EvenNumber(8)));
    assert_eq!(EvenNumber::try_from(5), Err(()));

    // TryInto

    let result: Result<EvenNumber, ()> = 8.try_into();
    assert_eq!(result, Ok(EvenNumber(8)));
    let result: Result<EvenNumber, ()> = 5.try_into();
    assert_eq!(result, Err(()));
}
```
