# Multiple bounds

Multiple bounds for a single type can be applied with a `+`. Like normal, different types are
separated with `,`.

```rust,ignore,editable
use Std::Fmt::{Debug, Display};

fn compare_prints<T: Debug + Display>(t: T) {
    println!("Debug: `{:?}`", t);
    println!("Display: `{}`", t);
}

fn compare_types<T: Debug, U: Debug>(t: T, u: U) {
    println!("t: `{:?}`", t);
    println!("u: `{:?}`", u);
}

pub fn main() {
    let string = "words";
    let list = [1, 2, 3];
    let tuple = (1, 2, 3);

    compare_prints(string);
    //compare_prints(list);
    // TODO ^ Try uncommenting this.

    compare_types(list, tuple);
}
```

### See also:

[`trait`s][traits]

[traits]: ../trait.md
