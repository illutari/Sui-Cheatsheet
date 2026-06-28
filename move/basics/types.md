# Primitive Types, Vectors, Strings, and Options

Move provides a small set of primitive types as building blocks. More complex data is built using structs and vectors. The standard library (implicitly available) adds `vector`, `Option`, `String`, and `ascii::String`.

## Variables and Assignment

```move
let <name>[: <type>] = <expr>;     // immutable by default
let mut <name>[: <type>] = <expr>; // mutable
```

Shadowing is allowed:

```move
let x: u8 = 42;
let x: u16 = 42; // new binding
```

## Booleans

`bool` with `true` / `false`. Type is usually inferred.

```move
let x = true;
let y = false;
```

Used heavily in control flow and assertions.

## Integer Types

Unsigned only:

| Type | Size | Common Use |
|------|------|------------|
| `u8` | 8-bit | Small counts, bytes, enums |
| `u16` | 16-bit | |
| `u32` | 32-bit | |
| `u64` | 64-bit | Balances, timestamps (default inference) |
| `u128` | 128-bit | Large arithmetic |
| `u256` | 256-bit | Crypto, very large numbers |

Literals: `42u8`, `42` (infers, usually u64).

### Arithmetic

Operations require matching types. Abort on overflow, underflow, div-by-zero.

| Op | Description | Aborts on |
|----|-------------|-----------|
| `+` | add | overflow |
| `-` | sub | underflow |
| `*` | mul | overflow |
| `/` | div (trunc) | div by 0 |
| `%` | mod | div by 0 |

### Casting

```move
let small: u8 = 255;
let big: u16 = (small as u16) + 1;
```

Parentheses often needed for precedence.

## Addresses

`address` is a 32-byte value (on Sui). Literals use `@`.

```move
let addr: address = @0x1;
let named = @std; // from [addresses] in Move.toml
```

Addresses identify accounts, packages, and objects. See the [Address section in the book](https://move-book.com/move-basics/address).

## Vectors

Native generic collection: `vector<T>`. Can hold any type (including other vectors, structs).

### Creation

```move
let empty: vector<u8> = vector[];
let nums: vector<u64> = vector[1, 2, 3];
let nested: vector<vector<u8>> = vector[ vector[1,2], vector[3] ];
```

### Common Operations (from `std::vector`, implicitly available)

```move
let mut v = vector[10u64, 20, 30];

v.length();     // 3
v.is_empty();   // false
v.push_back(40);
let last = v.pop_back(); // 40
v.remove(0);    // remove at index

// For non-`drop` elements, empty vectors must be explicitly destroyed
let mut nd = vector<NoDrop>[];
nd.destroy_empty();
```

See full `std::vector` for `borrow`, `swap`, `reverse`, `contains`, etc.

Vectors of `copy` types are themselves `copy` (if element is).

## Option

Generic `Option<Element>` represents "value or nothing". Implemented as a vector of length 0 or 1 (historical reason; not an enum yet in all contexts).

Implicitly available.

```move
use std::option::{Self, Option}; // usually not needed

let some: Option<u64> = option::some(42);
let none: Option<u64> = option::none();

assert!(some.is_some());
assert!(none.is_none());

let val = some.extract(); // takes it out, leaves empty Option
// or borrow: some.borrow()
```

Common in structs for optional fields (avoids magic empty values):

```move
public struct User has drop {
    name: String,
    middle_name: Option<String>, // explicit optional
}
```

`try_*` methods often return `Option` for fallible ops (e.g. `try_to_string`).

## Strings

No built-in string literal type; two wrappers in stdlib over `vector<u8>`:

- `std::string::String` — UTF-8 (preferred, has optimized native ops)
- `std::ascii::String` — ASCII only (lighter in some cases)

### Creating UTF-8 Strings

```move
use std::string::{Self, String};

let s1: String = string::utf8(b"Hello");
let s2 = b"Hello".to_string(); // convenient alias on vector<u8>

// Safe creation (returns Option)
let maybe = b"valid".try_to_string();
let bad = b"\xFF".try_to_string(); // is_none()
```

### Operations

```move
let mut s = b"Hello,".to_string();
let w = b" World!".to_string();

s.append(w);           // concat
s.sub_string(0, 5);    // "Hello" (copies slice; validates UTF-8 boundaries)
s.length();            // byte length (12)
s.is_empty();
let bytes: &vector<u8> = s.bytes(); // for low-level access
```

**Important limitations**:
- `length()` and indexing are in **bytes**, not characters (UTF-8 variable width 1-4 bytes).
- `sub_string` and similar abort on invalid char boundaries.
- No direct char iteration in base std (use external libs if needed).

ASCII module has similar API but restricted to ASCII.

In PTBs / tx inputs, byte vectors are often auto-converted to `String`.

## Best Practices

- Prefer `u64` unless you have a reason for smaller/larger (gas, clarity).
- Always handle overflow explicitly with casting or assertions when mixing sizes.
- Use `vector` for lists; `Option` for "maybe" instead of sentinel values.
- Prefer `String` over raw `vector<u8>` for user-facing text.
- For performance-critical string work, be aware of byte vs. char semantics.
- `Option` + `extract()` / `borrow()` is the idiomatic way to unwrap safely.
- Test edge cases around empty vectors of non-drop types and UTF-8 validity.

## Further Reading

- [Primitive Types](https://move-book.com/move-basics/primitive-types/)
- [Vector](https://move-book.com/move-basics/vector/)
- [Option](https://move-book.com/move-basics/option/)
- [String](https://move-book.com/move-basics/string/)
- [std::vector](https://docs.sui.io/references/framework/std/vector), [std::option](https://docs.sui.io/references/framework/std/option), [std::string](https://docs.sui.io/references/framework/std/string)
- Move Reference for full ops and casting rules.

---

*Previous: [Move Summary](../move_summary.md)*
*Next topics: [Structs](structs.md), [Functions](functions.md), [References](references.md), [Generics](generics.md), [Error Handling](errors.md) (see full list in [Move Summary](../move_summary.md))*
