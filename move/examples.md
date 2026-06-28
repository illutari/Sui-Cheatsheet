# Move Examples

Collection of small, self-contained Sui Move examples.

### Table of Contents

- [1. Hello World / Counter (Shared Object)](#1-hello-world--counter-shared-object)
- [2. Owned NFT with Display](#2-owned-nft-with-display)
- [3. Simple Capability Pattern](#3-simple-capability-pattern)
- [4. Using One-Time Witness (OTW) for Coin Creation](#4-using-one-time-witness-otw-for-coin-creation)
- [Next Steps](#next-steps)

## 1. Hello World / Counter (Shared Object)

```move
module examples::counter;

use sui::object::{Self, UID};
use sui::transfer;
use sui::tx_context::{Self, TxContext};

/// A simple shared counter.
public struct Counter has key {
    id: UID,
    value: u64,
}

fun init(ctx: &mut TxContext) {
    transfer::share_object(Counter {
        id: object::new(ctx),
        value: 0,
    });
}

public entry fun increment(c: &mut Counter) {
    c.value = c.value + 1;
}

public fun value(c: &Counter): u64 {
    c.value
}
```

**Usage after publish:**
```bash
sui client ptb --move-call $PKG::counter::increment @counter_id --gas-budget 10000000
```

*[Jump to top](#move-examples)*

## 2. Owned NFT with Display

```move
module examples::nft;

use sui::object::{Self, UID};
use sui::transfer;
use sui::tx_context::TxContext;
use sui::display;
use sui::package;

public struct MyNFT has key, store {
    id: UID,
    name: std::string::String,
    description: std::string::String,
}

public struct NFTMinter has key {
    id: UID,
}

fun init(ctx: &mut TxContext) {
    // One-time mint authority
    let minter = NFTMinter { id: object::new(ctx) };
    transfer::transfer(minter, tx_context::sender(ctx));

    // Display standard metadata
    let publisher = package::claim_for_testing(ctx); // in real: use Publisher object
    let display = display::new_with_fields<MyNFT>(
        &publisher,
        vector[
            std::string::utf8(b"name"),
            std::string::utf8(b"description"),
            std::string::utf8(b"image_url"),
        ],
        vector[
            std::string::utf8(b"{name}"),
            std::string::utf8(b"{description}"),
            std::string::utf8(b"https://example.com/nft/{id}"),
        ],
        ctx,
    );
    display::update_version(&mut display);
    transfer::public_freeze_object(display);
    // In real code, transfer Publisher or use one-time witness pattern.
}

public entry fun mint(
    _minter: &NFTMinter,
    name: std::string::String,
    description: std::string::String,
    recipient: address,
    ctx: &mut TxContext,
) {
    let nft = MyNFT {
        id: object::new(ctx),
        name,
        description,
    };
    transfer::public_transfer(nft, recipient);
}
```

*[Jump to top](#move-examples)*

## 3. Simple Capability Pattern

```move
module examples::admin;

use sui::object::{Self, UID};
use sui::transfer;
use sui::tx_context::TxContext;

public struct AdminCap has key, store {
    id: UID,
}

public struct Config has key {
    id: UID,
    fee: u64,
}

fun init(ctx: &mut TxContext) {
    let cap = AdminCap { id: object::new(ctx) };
    transfer::transfer(cap, tx_context::sender(ctx));

    let config = Config { id: object::new(ctx), fee: 100 };
    transfer::share_object(config);
}

public entry fun set_fee(
    _cap: &AdminCap,
    config: &mut Config,
    new_fee: u64,
) {
    config.fee = new_fee;
}
```

Callers must pass the `AdminCap` object to privileged functions.

*[Jump to top](#move-examples)*

## 4. Using One-Time Witness (OTW) for Coin Creation

(Abbreviated — see full Coin docs)

```move
module examples::my_coin;

use sui::coin;
use sui::url;

public struct MY_COIN has drop {}

fun init(witness: MY_COIN, ctx: &mut TxContext) {
    let (treasury, metadata) = coin::create_currency(
        witness,
        9,
        b"MYC",
        b"My Coin",
        b"",
        option::some(url::new_unsafe_from_bytes(b"https://...")),
        ctx,
    );
    transfer::public_freeze_object(metadata);
    transfer::public_transfer(treasury, tx_context::sender(ctx));
}
```

*[Jump to top](#move-examples)*

## Next Steps

- Combine with PTBs (`sui client ptb --move-call ...`)
- Add events with `sui::event::emit`
- Use dynamic fields for flexible storage: `sui::dynamic_field`, `sui::table`
- Explore the full [Move Book examples and patterns](https://move-book.com/)

---

*Previous: [Move Summary](move_summary.md)*
