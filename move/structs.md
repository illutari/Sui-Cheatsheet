# Move Structs and Abilities

Structs define custom data types. On Sui they become the basis for on-chain **objects** when they have the `key` ability.

## Defining a Struct

```move
public struct Counter has key, store {
    id: UID,           // required for objects (key ability)
    value: u64,
    owner: address,
}
```

- Fields are accessed by name (no positional).
- Structs are **nominal** (two structs with identical fields are different types).

## Creating (Packing) and Destructuring

```move
// Pack
let c = Counter {
    id: object::new(ctx),
    value: 0,
    owner: tx_context::sender(ctx),
};

// Destructure (move fields out)
let Counter { id, value, owner } = c;
// or ignore some
let Counter { value, .. } = c;
```

For `key` objects you rarely destructure the whole thing because the `UID` must be handled carefully.

## Abilities

Abilities are type-level permissions:

| Ability | Meaning | Common Use |
|---------|---------|------------|
| `copy`  | Values of this type can be copied (duplicated) | Small data, IDs sometimes |
| `drop`  | Values can be silently discarded at end of scope | Most non-resource data |
| `store` | Values can be stored inside other structs / objects | Almost everything that goes into objects |
| `key`   | The type can be a top-level Sui object (stored in global storage) | All on-chain objects |

**Important rules:**
- `key` implies `store` in practice for Sui objects.
- To store a struct inside an object you need `store`.
- Resources (things you don't want duplicated or lost) usually have **only `key` + `store`** and omit `copy`/`drop`.

```move
// Typical object
public struct Coin has key, store { ... }

// Pure data value (can be copied around inside tx)
public struct Stats has copy, drop, store {
    count: u64,
}
```

## Public vs. non-public structs

- `public struct` — can be constructed / destructured from other modules (if abilities allow).
- `struct` (private) — only the defining module can pack/unpack.

On Sui most object structs are declared `public struct` so other modules can read fields via getter functions (but construction is still controlled).

## Methods (receiver syntax)

```move
// In the module that defines the struct
public use fun value as Counter.value;

public fun value(c: &Counter): u64 {
    c.value
}

// Caller (any module that can see Counter)
let v = counter.value();   // nice syntax thanks to use fun
```

## Example: Simple Counter Struct

```move
module my_pkg::counter;

use sui::object::{Self, UID};
use sui::transfer;
use sui::tx_context::{Self, TxContext};

public struct Counter has key, store {
    id: UID,
    value: u64,
}

fun init(ctx: &mut TxContext) {
    let counter = Counter {
        id: object::new(ctx),
        value: 0,
    };
    transfer::share_object(counter);
}

public fun increment(c: &mut Counter) {
    c.value = c.value + 1;
}

public fun get(c: &Counter): u64 { c.value }
```

---

*Previous: [Modules](modules.md)*
*Next: [Functions](functions.md), [Objects](objects.md)*
*See also: [References & the Borrow Checker](references.md) (deep dive on & / &mut, ownership rules enforced by the checker, and how abilities interact with borrowing)*
