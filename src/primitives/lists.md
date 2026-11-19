# Lists

A list is an ordered collection of elements of the same type `T`. Lists are created
using brackets `[]`. The number of elements is said to be the *length* of the list.

A list is a recursive data structure: `List[T] ::= Empty | Cons(T, List[T])`.
In other words, a *list* is either the empty list `[]`, or consists of a *head* (first element)
and a *tail* (remainder of the list) (see the [Erlang documentation on lists][erlang list]).

Note that lists on BEAM platform *are not random-access*. This means that to get Nth element
of a list, you need to traverse the previous N-1 elements. For this reason, Patina does not
provide a bracket access operator (ex. `lst[0]`).

```rust,editable,ignore
pub fn main() {
    // List (type signature is superfluous).
    let xs: [int] = [1, 2, 3, 4, 5];

    // Indexing starts at 0.
    // `.get(n)` can be used to traverse to and return the nth element of the list
    // as an `Option`.
    match xs.get(0) {
        Some(xval) => println!("elem 0: {}", xval),
        None => println!("the list is empty!"),
    }

    // `len` returns the count of elements in the list.
    // Internally, it does that by traversing the list and counting all elements.
    println!("Number of elements in array: {}", xs.len());

    // The head and tail of a list can be retrieved with `head` and `tail`
    // `head` is an `Option`, and is None for empty lists; `tail` always succeeds.
    println!("First element of the list: {}", xs.head());
    println!("Rest of the list: {:?}", xs.tail());

    // Lists can be prepended in constant time using the following syntax:
    let more_xs = [-1, 0, ..xs];
    // This creates a new list, but the tail is not duplicated - `xs`
    // and `more_xs.tail().tail()` are internally the same VM object.
    println!("more_xs.tail().tail() is equal to xs: {}", more_xs.tail().tail() == xs);

    // Lists can also be destructured using a similar syntax
    let [first, second, third, rest @ ..] = more_xs else { return };
    println!("Sum of first three elements is {}", first + second + third);
    println!("...and the rest: {:?}", rest);
}
```

[erlang list]: https://www.erlang.org/doc/system/data_types.html#list
