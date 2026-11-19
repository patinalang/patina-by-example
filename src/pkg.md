# Packages

A package (`pkg`) is the basic unit of distribution for Patina libraries.
It's what you publish to the package manager like [Hex.pm][hex], what you specify in your
dependencies list, and what you reference when calling these dependencies.

## Creating a package

`patina new my_json_lib` creates a new package in the `my_json_lib` directory:

```text
my_json_lib/
├── Patina.toml
├── rebar.config.script
└── src/
    ├── my_json_lib.pt
    └── my_json_lib.app.src.script
```

The two `.script` files make the package a regular [rebar3]-compatible project.

## Naming conventions

A package has three top-level names:

* **The package name itself**: this is the `snake_case` name that Patina code uses to reference
  the package, e.g. `my_json_lib` in `my_json_lib::decoder::parse(input)`.
  It must be a valid identifier, and `core` and `std` are reserved.
  For most packages, this is the only name that's set explicitly.

* **The OTP Application name**: every package is compiled into an [OTP application], which defaults
  to the _package name_ by default.

* **The Hex package name**: if the package is published to [Hex.pm][hex], this is the name
* it's published under. It defaults to the _OTP Application name_.

## Package manifest

The root directory of a Patina package must contain a `Patina.toml` file:

```toml
[package]
name = "my_json_lib"
version = "0.1.0"
description = "A JSON library" # optional
otp_app_name = "my_json_lib"   # optional and discouraged; defaults to the package name
path = "src/my_json_lib.pt"    # optional; defaults to src/<name>,pt

[dependencies]
# A package from Hex.pm, with a version requirement.
my_utils = "~> 1.0"
# A package from a git repository, optionally at a branch, tag, or commit.
my_parser = { git = "https://example.com/my_parser.git", ref = "v0.2.0" }
# A package from a local directory.
my_helpers = { path = "../my_helpers" }
```

## Package file

The _package file_ is the root source file of the package. By default, its path relative
to `Patina.toml` is `src/<pkg name>.pt`, but it can be set in `Patina.toml`.
Patina compiler will start reading from that file, and, when it encounters `mod` declarations,
contents of the module files will be inserted in their place. Files that are not referenced
by the tree rooted in the _package file_ will not be read or compiled.

[hex]: https://hex.pm
[OTP application]: https://www.erlang.org/doc/system/applications.html
[rebar3]: https://rebar3.org
