# let-else

With `let`-`else`, a refutable pattern can match and bind variables
in the surrounding scope like a normal `let`, or else diverge (e.g. `break`,
`return`, `panic!`) when the pattern doesn't match.

```rust,ignore
use str::FromStr;

fn get_count_item(s: str) -> (int, str) {
    let it = s.split(' ');
    let (it, first) = it.next();
    let (it, second) = it.next();

    let (Some(count_str), Some(item)) = (first, second) else {
        panic!("Can't segment count item pair: '{s}'");
    };
    let Ok(count) = int::from_str(count_str) else {
        panic!("Can't parse integer: '{count_str}'");
    };
    (count, item)
}

pub fn main() {
    assert_eq!(get_count_item("3 chairs"), (3, "chairs"));
}
```

### See also:

[option][option], [match][match], [if let][if_let].

[match]: ./match.md
[if_let]: ./if_let.md
[option]: ../error/option_unwrap.md
