# Move Language Quick Reference (Sui)

Move is a secure, resource-oriented programming language for writing smart contracts on Sui (and other chains). It emphasizes **linear types** (resources) that cannot be copied or dropped accidentally, making it excellent for digital assets.

Sui extends Move with a powerful **object-centric storage model**, fast-path execution for owned objects, and rich framework modules.

**Run `sui move build` / `sui move test` from your package root** (directory with `Move.toml`).

See the [Sui Move CLI cheatsheet](../cli/sui/sui_move.md) for compiler commands.

### Table of Contents

- [Core Concepts](#core-concepts)
- [Basic Topics](#basic-topics)
- [Sui Object Model](#sui-object-model)
- [Common Patterns](#common-patterns)
- [Quick Syntax Examples](#quick-syntax-examples)
- [Official Resources](#official-resources)

## Core Concepts

| Concept | Description | Link |
|---------|-------------|------|
| **Package** | Unit of deployment. Contains modules + dependencies. Published as immutable object on-chain. | [Packages](../concepts but see below) |
| **Module** | Basic unit of code. Declared as `module address::name;`. All items private by default. | [Modules](modules.md) |
| **Struct** | Custom type. Can have abilities that control copy/drop/store/key behavior. | [Structs](structs.md) |
| **Resource** | A struct with the `key` ability (on Sui) representing an on-chain object with unique ID. | [Objects](objects.md) |
| **Abilities** | `copy`, `drop`, `store`, `key` — declare what operations are allowed on a type. | [Structs](structs.md) |
| **Function** | Logic unit. `fun`, `public`, `public(package)`, `entry`, `public entry`. | [Functions](functions.md) |

## Basic Topics

- [Modules, Imports, and Visibility](modules.md)
- [Primitive Types, Vectors, Strings, Options](types.md) *(planned)*
- [Structs, Fields, and Packing/Unpacking](structs.md)
- [Functions, Methods, and Control Flow](functions.md)
- [References (`&`, `&mut`), Ownership, and Borrowing](references.md) *(planned)*
- [Generics and Constraints](generics.md) *(planned)*
- [Constants, Asserts/Aborts, and Error Handling](errors.md) *(planned)*

## Sui Object Model

The key differentiator of Sui Move:

- Objects have a unique `UID` (created with `object::new`).
- Ownership: Owned (by address or object), Shared, Immutable (frozen).
- `key` ability + `store` ability required for most storage/transfer ops.
- Fast path for owned objects (no consensus needed for transfers).
- Transfer functions live in `sui::transfer`.
- `init` module initializer runs exactly once on publish.

See dedicated guide: [Objects and Storage](objects.md)

## Common Patterns

- **Capability** (admin tokens as objects)
- **Witness / One-Time Witness (OTW)**
- **Hot Potato** (non-droppable types that must be consumed)
- **Wrapper** types
- **Dynamic Fields / Tables** (Bag, Table, etc. for heterogeneous or large collections)
- **Events** (`sui::event`)
- **Display** standard for rich metadata
- **Publisher** for package authority

See: [Patterns](patterns.md) *(planned)*

## Quick Syntax Examples

### Module + Struct (with abilities)
```move
module my_pkg::counter;

public struct Counter has key, store {
    id: UID,
    value: u64,
}

fun init(ctx: &mut TxContext) {
    transfer::share_object(Counter {
        id: object::new(ctx),
        value: 0,
    });
}
```

### Entry function + references
```move
public entry fun increment(c: &mut Counter) {
    c.value = c.value + 1;
}
```

### Transfer / freeze / share
```move
// transfer ownership
transfer::transfer(obj, recipient);

// share for anyone to use (consensus path)
transfer::share_object(obj);

// make immutable (e.g. for a published package's policy)
transfer::freeze_object(obj);
```

See full examples in the topics below and in the [Move Book examples](https://move-book.com/).

## Official Resources

- The Move Book (excellent, Sui-focused): https://move-book.com/
- Sui Move Concepts: https://docs.sui.io/develop/write-move/sui-move-concepts
- Sui Docs - Write Move: https://docs.sui.io/develop/write-move
- Move Reference (formal): https://move-language.github.io/move/
- Sui Framework source (on-chain stdlib): https://github.com/MystenLabs/sui/tree/main/crates/sui-framework/packages

For the most up-to-date language features run `sui move build --help` or consult the edition notes (Move 2024).

---

*Related: [Sui Move CLI Cheatsheet](../cli/sui/sui_move.md)*

---

Topics implemented: [modules](modules.md) • [structs](structs.md) • [functions](functions.md) • [objects](objects.md) • [examples](examples.md)
