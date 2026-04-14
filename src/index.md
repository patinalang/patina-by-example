# Patina by Example

[Patina][patina] is a statically typed programming language for the [BEAM][beam] virtual machine,
focusing on type safety and easy interop with broader [Elixir][elixir] ecosystem. Its syntax,
type semantics, and much of its compiler code, is directly derived from [Rust][rust].

Patina by Example is a collection of examples that illustrate various Patina concepts.
It is a fork of the excellent [Rust By Example (RBE)][rbe], with many parts unchanged - where Patina
and Rust syntax match exactly.

Now let's begin!

- [Hello World](hello.md) - Start with a traditional Hello World program.

- [Primitives](primitives.md) - Learn about signed integers, unsigned integers and other primitives.

- [Custom Types](custom_types.md) - `struct` and `enum`.

- [Variable Bindings](variable_bindings.md) - mutable bindings, scope, shadowing.

- [Types](types.md) - Learn about changing and defining types.

- [Conversion](conversion.md) - Convert between different types, such as strings, integers, and floats.

- [Expressions](expression.md) - Learn about Expressions & how to use them.

- [Flow of Control](flow_control.md) - `if`/`else`, `for`, and others.

- [Functions](fn.md) - Learn about Methods, Closures and Higher Order Functions.

- [Modules](mod.md) - Organize code using modules.

- [Packages](pkg.md) - Organize modules into distributable packages.

- [Attributes](attribute.md) - An attribute is metadata applied to some module or item.

- [Generics](generics.md) - Learn about writing a function or data type which can work for multiple types of arguments.

- [Traits](trait.md) - A trait is a collection of methods defined for an unknown type: `Self`

- [Macros](macros.md) - Macros are a way of writing code that writes other code, which is known as metaprogramming.

- [Error handling](error.md) - Learn Patina way of handling failures.

- [Unsafe Operations](unsafe.md) - Learn about entering a block of unsafe operations.

[patina]: https://patina.pt/
[beam]: https://en.wikipedia.org/wiki/BEAM_(Erlang_virtual_machine)
[elixir]: https://elixir-lang.org/
[rust]: https://www.rust-lang.org/
[rbe]: https://github.com/rust-lang/rust-by-example
