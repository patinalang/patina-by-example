# Functions as input and output parameters

## Input parameters

Closures and functions may both be used as function arguments. Simple function references are typed
with `fnref()`, while closures are typed with `fn()` - with former implicitly convertible
to the latter.

```rust,ignore,editable
fn call_me_ref(f: fnref(int) -> int) {
    f(1);
}

fn call_me(f: fn(int) -> int) {
    f(2);
}

fn function(x: int) -> int {
    x + 1
}

pub fn main() {
    let closure = |x| x + 42;

    call_me(function);
    call_me(closure);

    call_me_ref(function);
    // call_me_ref(closure);
    // ^ ERROR! `fn(int) -> int` cannot be used as `fnref(int) -> int`
}
```

## Output parameters

Just as `fn` and `fnref` can be used as input parameters, they can also be returned from other
functions.

```rust,ignore,editable
fn lazy_add(fx: fn() -> int, fy: fn() -> int) -> fn() -> int {
    || fx() + fy()
}

fn add(x: int, y: int) -> int {
    x + y
}

fn mul(x: int, y: int) -> int {
    x * y
}

enum MathOp {
    Add,
    Mul,
}

fn choose_op(op: MathOp) -> fnref(int, int) -> int {
    match op {
        MathOp::Add => add,
        MathOp::Mul => mul,
    }
}

pub fn main() {
    let fx = || {
        println!("fx evaluated!");
        3
    };

    let fy = || {
        println!("fy evaluated!");
        5
    };

    // won't print anything
    let lazy_result = lazy_add(fx, fy);

    let op = choose_op(MathOp::Add);
    println!("{} op {} = {}", 3, 5, op(3, 5))
}
```
