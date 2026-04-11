# Modules

Patina provides a powerful module system that can be used to hierarchically split
code in logical units (modules), and manage visibility (public/private) between
them.

A Patina module is a collection of items: functions, structs, traits, `impl` blocks,
and even other modules.

Every Patina module uniquely maps to a BEAM module, but not every Patina-created BEAM module
maps to a Patina module. For example, every `struct` and `enum` gets its own BEAM module.
