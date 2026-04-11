# Associated functions & Methods

Some functions are connected to a particular type. These come in two forms:
associated functions, and methods. Associated functions are functions that
are defined on a type generally, while methods are associated functions that are
called on a particular instance of a type.

```rust,ignore,editable
struct Point {
    x: float,
    y: float,
}

// Implementation block, all `Point` associated functions & methods go in here
impl Point {
    // This is an "associated function" because this function is associated with
    // a particular type, that is, Point.
    //
    // Associated functions don't need to be called with an instance.
    // These functions are generally used like constructors.
    fn origin() -> Point {
        Point { x: 0.0, y: 0.0 }
    }

    // Another associated function, taking two arguments:
    fn new(x: float, y: float) -> Point {
        Point { x: x, y: y }
    }
}

struct Rectangle {
    p1: Point,
    p2: Point,
}

impl Rectangle {
    // This is a method
    // `self` is sugar for `self: Self`, where `Self` is the type of the
    // caller object. In this case `Self` = `Rectangle`
    fn area(self) -> float {
        // `self` gives access to the struct fields via the dot operator
        let Point { x: x1, y: y1 } = self.p1;
        let Point { x: x2, y: y2 } = self.p2;

        // `abs` is a `float` method that returns the absolute value of the
        // caller
        ((x1 - x2) * (y1 - y2)).abs()
    }

    fn perimeter(self) -> float {
        let Point { x: x1, y: y1 } = self.p1;
        let Point { x: x2, y: y2 } = self.p2;

        2.0 * ((x1 - x2).abs() + (y1 - y2).abs())
    }

    // Values in BEAM are immutable, so traditionally "mut" methods return
    // the new value instead.
    fn translate(self, x: float, y: float) -> Self {
        Rectangle {
            p1: Point { x: self.p1.x + x, y: self.p1.y + y },
            p2: Point { x: self.p2.x + x, y: self.p2.y + y },
        }
    }
}

pub fn main() {
    let rectangle = Rectangle {
        // Associated functions are called using double colons
        p1: Point::origin(),
        p2: Point::new(3.0, 4.0),
    };

    // Methods are called using the dot operator
    // Note that the first argument `self` is implicitly passed, i.e.
    // `rectangle.perimeter()` === `Rectangle::perimeter(rectangle)`
    println!("Rectangle perimeter: {}", rectangle.perimeter());
    println!("Rectangle area: {}", rectangle.area());

    // `mut` keyword lets us rebind the variable in the next statement.
    let mut square = Rectangle {
        p1: Point::origin(),
        p2: Point::new(1.0, 1.0),
    };

    square = square.translate(1.0, 1.0);
}
```
