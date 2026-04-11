# Disambiguating overlapping traits

A type can implement many different traits. What if two traits both require
the same name for a function? For example, many traits might have a method
named `get()`. They might even have different return types!

Good news: because each trait implementation gets its own `impl` block, it's
clear which trait's `get` method you're implementing.

What about when it comes time to _call_ those methods? To disambiguate between
them, we have to use Fully Qualified Syntax.

```rust,ignore,editable
trait UsernameWidget {
    // Get the selected username out of this widget
    fn get(self) -> str;
}

trait AgeWidget {
    // Get the selected age out of this widget
    fn get(self) -> int;
}

// A form with both a UsernameWidget and an AgeWidget
struct Form {
    username: str,
    age: int,
}

impl UsernameWidget for Form {
    fn get(self) -> str {
        self.username
    }
}

impl AgeWidget for Form {
    fn get(self) -> int {
        self.age
    }
}

pub fn main() {
    let form = Form {
        username: "rustacean",
        age: 28,
    };

    // If you uncomment this line, you'll get an error saying
    // "multiple `get` found". Because, after all, there are multiple methods
    // named `get`.
    // println!("{}", form.get());

    let username = <Form as UsernameWidget>::get(form);
    assert_eq!("rustacean", username);
    let age = <Form as AgeWidget>::get(form);
    assert_eq!(28, age);
}
```
