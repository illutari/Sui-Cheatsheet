# Move Modules

Modules are the fundamental building blocks of Move code. A module groups related types, functions, and constants. All members are **private by default** (only visible inside the module).

## Module Declaration

```move
// Preferred (label syntax, Move 2024+)
module my_pkg::my_module;

// Old block syntax (only needed for >1 module per file - discouraged)
module my_pkg::another_module {
    // ...
}
```

- The address (`my_pkg` or `0x...`) comes from the `[addresses]` section in `Move.toml` or a literal.
- Module name is `snake_case`.
- Convention: one module per `.move` file in `sources/`, filename matches module name (e.g. `my_module.move`).

## Module Members

Typical members inside a module:

```move
module book::example;

// Use / import
use std::string::{Self, String};
use sui::object::UID;
use sui::transfer;

// Constant
const MAX_VALUE: u64 = 1000;

// Struct
public struct MyStruct has key, store {
    id: UID,
    value: u64,
}

// Public function (callable from other modules)
public fun create(ctx: &mut TxContext): MyStruct {
    MyStruct { id: object::new(ctx), value: 0 }
}

// Entry function (callable directly in PTB / transaction)
public entry fun set_value(obj: &mut MyStruct, v: u64) {
    obj.value = v;
}

// Private helper (only inside this module)
fun validate(v: u64) {
    assert!(v <= MAX_VALUE, ETooLarge);
}
```

## Visibility

| Modifier          | Callable from                  | Notes |
|-------------------|--------------------------------|-------|
| (no modifier)     | Same module only               | Private |
| `public`          | Any module                     | Most common for APIs |
| `public(package)` | Modules in the same package    | Sui/Move 2024 feature |
| `entry`           | Transaction entry point        | Can be combined: `public entry` |
| `public entry`    | Both external + tx entry       | Frequently used |

## Imports (`use`)

```move
// Full path
use std::vector;

// Rename on import
use std::string as str;

// Import specific members
use std::string::{String, utf8};

// Use fun for method syntax (receiver style)
public use fun my_helper as MyStruct.helper;
```

See also: [Importing Modules in the Move Book](https://move-book.com/move-basics/importing-modules.md)

## Multiple Modules / File Organization

- One module per file is strongly recommended.
- Use subdirectories under `sources/` for larger packages.
- Dependencies are declared in `Move.toml`.

## Best Practices

- Keep modules focused (single responsibility).
- Use `public(package)` for package-internal APIs.
- Document public functions with `///` doc comments.
- Name modules descriptively in `snake_case`.

## Quick Example: Minimal Module

```move
module my_pkg::hello;

use std::string::String;

/// A simple greeting.
public fun greet(name: String): String {
    // ...
    name // placeholder
}
```

---

*Previous: [Move Summary](../move_summary.md)*
*Next topics: [Structs](structs.md), [Functions](functions.md), [References](references.md), [Generics](generics.md), [Error Handling](errors.md)*
