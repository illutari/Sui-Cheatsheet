# Sui Object Model in Move

Sui's object model is one of the biggest differences from account-based Move blockchains. Almost everything of value is represented as an **object** with a unique ID, clear ownership, and efficient access paths.

## What is a Sui Object?

Any struct with the `key` ability is a potential object:

```move
public struct MyNFT has key, store {
    id: UID,
    name: String,
    // other fields...
}
```

Required:
- An `id: UID` field (created via `object::new(ctx)`).
- The `key` ability (makes it a first-class object).
- Usually `store` so it can be transferred / stored inside other objects.

## Creating Objects

```move
fun init(ctx: &mut TxContext) {
    let nft = MyNFT {
        id: object::new(ctx),
        name: string::utf8(b"My first NFT"),
    };
    transfer::transfer(nft, tx_context::sender(ctx)); // owned by sender
}
```

## Ownership Types

| Ownership | Description | Consensus? | Transfer |
|-----------|-------------|------------|----------|
| **Owned by address** | Single owner (most common for user assets) | No (fast path) | `transfer::transfer` or `public_transfer` |
| **Owned by object** | Child object (Transfer to Object / TTO) | Depends on parent | Special receive flow |
| **Shared** | Mutable by anyone (e.g. AMMs, global state) | Yes | `transfer::share_object` |
| **Immutable / Frozen** | Read-only forever (e.g. published packages, policies) | No | `transfer::freeze_object` |

## Storage Functions (`sui::transfer`)

```move
// Give ownership to an address
transfer::public_transfer(obj, recipient_addr);

// Share (anyone can call entry functions on it)
transfer::public_share_object(obj);

// Freeze forever (great for capabilities you want to make immutable)
transfer::public_freeze_object(obj);

// Receive an object sent to another object (advanced TTO)
let received = transfer::receive(&mut parent, ticket);
```

## UID and ID

```move
use sui::object::{Self, UID, ID};

let uid = object::new(ctx);
let id: ID = object::uid_to_inner(&uid);
object::delete(uid); // only when object is destroyed
```

- Every object has a globally unique ID.
- `ID` is copyable and can be used as a key in tables/dynamic fields.

## The `key` + `store` Abilities (recap)

- `key`: Allows the type to be a top-level stored object.
- `store`: Allows the type (or values of it) to live inside other objects, or be transferred publicly.

Most Sui objects are declared:

```move
public struct Thing has key, store { ... }
```

## Receiving Objects (TTO - Transfer To Object)

```move
public struct Parent has key {
    id: UID,
    // ...
}

// In a function that receives from another object
public fun receive_from_parent(
    parent: &mut Parent,
    ticket: Receiving<MyChild>,
) {
    let child: MyChild = transfer::receive(&mut parent.id, ticket);
    // ...
}
```

## Best Practices

- Prefer owned objects for user-controlled assets (fast, private).
- Use shared objects only when you truly need multi-writer access.
- Always protect privileged operations on shared objects with capability objects.
- Freeze objects you never want to change again (e.g. upgrade policies, Display objects).
- Delete UIDs only when the object is being fully removed from storage.

## Example: Minimal Owned Object

```move
module my_pkg::item;

use sui::object::{Self, UID};
use sui::transfer;
use sui::tx_context::TxContext;

public struct Item has key, store {
    id: UID,
    power: u64,
}

public fun mint(ctx: &mut TxContext): Item {
    Item { id: object::new(ctx), power: 10 }
}

public entry fun transfer(item: Item, to: address) {
    transfer::public_transfer(item, to);
}
```

See also the full [Object Model chapter in the Move Book](https://move-book.com/object/index.md).

---

*Previous: [Functions](functions.md)*
*Related: [Move Summary](move_summary.md), [Structs](structs.md)*
