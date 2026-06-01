# To and from `str`s

## Converting to `str`

To convert any type to a `str` is as simple as implementing the `ToStr`
trait for the type. Rather than doing so directly, you should implement the
`Fmt::Display` trait which automatically provides `ToStr` and
also allows printing the type.

```rust,ignore,editable
use Std::Fmt;

struct Circle {
    radius: int
}

impl Fmt::Display for Circle {
    fn fmt(self, f: Fmt::Formatter) -> Fmt::Result {
        write!(f, "Circle of radius {}", self.radius)
    }
}

pub fn main() {
    let circle = Circle { radius: 6 };
    println!("{}", circle.to_str());
}
```

## Parsing a `str`

It's useful to convert strings into many types, but one of the more common string
operations is to convert them from string to number. The idiomatic approach to
this is to use the `parse` function and either to arrange for type inference or
to specify the type to parse using the 'turbofish' syntax. Both alternatives are
shown in the following example.

This will convert the string into the type specified as long as the `FromStr`
trait is implemented for that type. This is implemented for numerous types
within the standard library.

```rust,ignore,editable
pub fn main() {
    let parsed: int = "5".parse().unwrap();
    let turbo_parsed = "10".parse::<int>().unwrap();

    let sum = parsed + turbo_parsed;
    println!("Sum: {:?}", sum);
}
```

To obtain this functionality on a user defined type simply implement the
`FromStr` trait for that type.

```rust,ignore,editable
use Std::Num::ParseIntError;
use str::FromStr;

#[derive(Debug)]
struct Circle {
    radius: int,
}

impl FromStr for Circle {
    type Err = ParseIntError;
    fn from_str(s: str) -> Result<Self, Self::Err> {
        match s.trim().parse() {
            Ok(num) => Ok(Circle{ radius: num }),
            Err(e) => Err(e),
        }
    }
}

pub fn main() {
    let radius = "    3 ";
    let circle: Circle = radius.parse().unwrap();
    println!("{:?}", circle);
}
```
