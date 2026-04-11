# Visibility

By default, the items in a module have private visibility, but this can be
overridden with the `pub` modifier. Only the public items of a module can be
accessed from outside the module scope.

```rust,ignore,editable
// A module named `MyMod`.
// Unlike Rust, Patina uses CamelCase for module naming.
// This makes calling Patina code from Elixir significantly more ergonomic.
mod MyMod {
    // On BEAM side, module `MyMod` will be saved as `Patina.MyPkg.MyMod.beam` .
    // However, this is configurable so you can name your BEAM module anything.
    // For example, to replace `MyMod` written in Elixir:
    #![beam_module("Elixir.MyMod")]

    // Items in modules default to private visibility.
    fn private_function() {
        println!("called `MyMod::private_function()`");
    }

    // Use the `pub` modifier to override default visibility.
    pub fn function() {
        println!("called `MyMod::function()`");
    }

    // Items can access other items in the same module,
    // even when private.
    pub fn indirect_access() {
        print!("called `MyMod::indirect_access()`, that\n> ");
        private_function();
    }

    // Modules can also be nested
    pub mod Nested {
        pub fn function() {
            println!("called `MyMod::Nested::function()`");
        }

        #[allow(dead_code)]
        fn private_function() {
            println!("called `MyMod::Nested::private_function()`");
        }

        // Functions declared using `pub(in path)` syntax are only visible
        // within the given path. `path` must be a parent or ancestor module.
        pub(in pkg::MyMod) fn public_function_in_my_mod() {
            print!("called `MyMod::Nested::public_function_in_my_mod()`, that\n> ");
            public_function_in_nested();
        }

        // Functions declared using `pub(self)` syntax are only visible within
        // the current module, which is the same as leaving them private - except
        // they *will* be exported by the generated BEAM module.
        pub(self) fn public_function_in_nested() {
            println!("called `MyMod::Nested::public_function_in_nested()`");
        }

        // Functions declared using `pub(super)` syntax are only visible within
        // the parent module. The same caveats apply - these functions are still
        // exported in the BEAM module.
        pub(super) fn public_function_in_super_mod() {
            println!("called `MyMod::Nested::public_function_in_super_mod()`");
        }
    }

    pub fn call_public_function_in_my_mod() {
        print!("called `MyMod::call_public_function_in_my_mod()`, that\n> ");
        Nested::public_function_in_my_mod();
        print!("> ");
        Nested::public_function_in_super_mod();
    }

    // pub(pkg) makes functions visible only within the current package.
    pub(pkg) fn public_function_in_pkg() {
        println!("called `MyMod::public_function_in_pkg()`");
    }

    // Nested modules follow the same rules for visibility
    mod private_nested {
        #[allow(dead_code)]
        pub fn function() {
            println!("called `MyMod::private_Nested::function()`");
        }

        // Private parent items will still restrict the visibility of a child item,
        // even if it is declared as visible within a bigger scope.
        #[allow(dead_code)]
        pub(pkg) fn restricted_function() {
            println!("called `MyMod::private_Nested::restricted_function()`");
        }
    }
}

fn function() {
    println!("called `function()`");
}

pub fn main() {
    // Modules allow disambiguation between items that have the same name.
    function();
    MyMod::function();

    // Public items, including those inside nested modules, can be
    // accessed from outside the parent module.
    MyMod::indirect_access();
    MyMod::Nested::function();
    MyMod::call_public_function_in_my_mod();

    // pub(pkg) items can be called from anywhere in the same pkg
    MyMod::public_function_in_pkg();

    // pub(in path) items can only be called from within the module specified
    // Error! function `public_function_in_my_mod` is private
    //MyMod::Nested::public_function_in_my_mod();
    // TODO ^ Try uncommenting this line

    // Private items of a module cannot be directly accessed, even if
    // nested in a public module:

    // Error! `private_function` is private
    //MyMod::private_function();
    // TODO ^ Try uncommenting this line

    // Error! `private_function` is private
    //MyMod::Nested::private_function();
    // TODO ^ Try uncommenting this line

    // Error! `private_nested` is a private module
    //MyMod::private_Nested::function();
    // TODO ^ Try uncommenting this line

    // Error! `private_nested` is a private module
    //MyMod::private_Nested::restricted_function();
    // TODO ^ Try uncommenting this line
}
```

## Visibility within BEAM

Non-`pub` functions are simply not exported from the generated BEAM modules.

Otherwise, all `pub` functions are exported - whether `pub(self)`, `pub(super)`, `pub(pkg)`,
or any other. In these cases, Patina enforces visibility locally.
