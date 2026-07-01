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
- [Terminology](#terminology)
- [Official Resources](#official-resources)

## Core Concepts

| Concept | Description | Link |
|---------|-------------|------|
| **Package** | Unit of deployment. Contains modules + dependencies. Published as immutable object on-chain. | [Packages](../concepts but see below) |
| **Module** | Basic unit of code. Declared as `module address::name;`. All items private by default. | [Modules](basics/modules.md) |
| **Struct** | Custom type. Can have abilities that control copy/drop/store/key behavior. | [Structs](basics/structs.md) |
| **Resource** | A struct with the `key` ability (on Sui) representing an on-chain object with unique ID. | [Objects](basics/objects.md) |
| **Abilities** | `copy`, `drop`, `store`, `key` — declare what operations are allowed on a type. | [Structs](basics/structs.md) |
| **Function** | Logic unit. `fun`, `public`, `public(package)`, `entry`, `public entry`. | [Functions](basics/functions.md) |
| **References & Borrow Checker** | `&T`, `&mut T`, ownership rules, move vs borrow, the static checker that enforces safety. | [References & Borrow Checker](basics/references.md) |

## Basic Topics

- [Modules, Imports, and Visibility](basics/modules.md)
- [Primitive Types, Vectors, Strings, Options](basics/types.md)
- [Structs, Fields, and Packing/Unpacking](basics/structs.md)
- [Functions, Methods, and Control Flow](basics/functions.md)
- [References (`&`, `&mut`), Ownership, and the Borrow Checker](basics/references.md)
- [Generics and Constraints](basics/generics.md)
- [Constants, Asserts/Aborts, and Error Handling](basics/errors.md)

## Sui Object Model

The key differentiator of Sui Move:

- Objects have a unique `UID` (created with `object::new`).
- Ownership: Owned (by address or object), Shared, Immutable (frozen).
- `key` ability + `store` ability required for most storage/transfer ops.
- Fast path for owned objects (no consensus needed for transfers).
- Transfer functions live in `sui::transfer`.
- `init` module initializer runs exactly once on publish.

See dedicated guide: [Objects and Storage](basics/objects.md)

## Common Patterns

- **Capability** (admin tokens as objects)
- **Witness / One-Time Witness (OTW)**
- **Hot Potato** (non-droppable types that must be consumed)
- **Wrapper** types
- **Dynamic Fields / Tables** (Bag, Table, etc. for heterogeneous or large collections)
- **Events** (`sui::event`)
- **Display** standard for rich metadata
- **Publisher** for package authority

See: [Patterns](basics/patterns.md) *(planned)*

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

## Terminology

This table maps common Move (Sui) terminology to concepts in more familiar languages like Java and Python. Move's design (inspired by Rust's ownership + linear types) emphasizes resource safety and explicit ownership, which differs from garbage-collected languages.

| Move Term | Definition in Move | Java Equivalent | Python Equivalent | Notes / Key Differences |
|-----------|--------------------|-----------------|-------------------|-------------------------|
| **Package** | Unit of deployment containing one or more modules (with `Move.toml`). Published as an immutable on-chain object. | Maven/Gradle project, JAR artifact, or JPMS module | Python package (directory with `pyproject.toml` / `setup.py`, or installed via pip) | Move packages are on-chain and immutable (but upgradable via capabilities). They define logic; state lives in objects. |
| **Module** | Compilation and encapsulation unit. Declares structs, functions, and constants. Fields of structs are private to the defining module by default. | Java class (or package of classes); visibility via `private`/`public` | Python module (`.py` file) or class with name mangling for "privacy" | Modules in Move strictly control access (like private-by-default in a Java class). A module can define multiple structs (unlike one public class per file in Java). |
| **Struct** | User-defined data type with named fields. Can be "plain" (in-memory) or a "resource" (on-chain object). | Java `class`, `record`, or POJO; `struct` in C | Python `class`, `@dataclass`, or `namedtuple` / `TypedDict` | Move structs are primarily data containers. Behavior (methods) lives in the module. No inheritance (use composition or generics). |
| **Resource** | A `struct` with the `key` ability (plus `store`), representing a unique on-chain asset/object that cannot be duplicated or lost accidentally. | No direct equivalent. Manual tracking with IDs + finalizers / try-with-resources; or DB entities | No direct. Use unique IDs + context managers (`__enter__`/`__exit__`), or libraries for "must-use" values | Move enforces linearity at the type system level (inspired by linear logic). Prevents bugs like double-spend or lost assets by default. |
| **Ability** (`copy`, `drop`, `store`, `key`) | Compile-time "traits" that control allowed operations on a type (copy values, drop/destroy, store inside other types, act as top-level on-chain object key). | Marker interfaces or annotations (e.g., `Serializable`, `Cloneable`); `final` / immutability patterns | Protocols / ABCs, dunder methods (`__copy__`, `__del__`, contextlib); no direct "key" concept | Abilities are opt-in and very specific. E.g., without `copy`, values are moved by default (Rust-like). `key` + `store` makes something a Sui object. |
| **Function** (`fun`, `public`, `public(package)`, `entry`) | Logic defined in a module. Visibility is module-private by default. `entry` functions are callable directly from transactions/PTBs. | Java method (`public`, package-private, etc.); `static` methods for module-like | Python function or method; module-level vs. classmethod | Move functions are "free" in the module (not attached to instances like methods). `entry` is like a public API endpoint or `main` for txs. |
| **Entry Function** | A `public entry fun` that can be the root of a user transaction (called from outside via PTB or client). | Public static method or controller endpoint | Top-level function exposed via CLI / API | Strict rules in older Move (no refs in some cases); now flexible but still the on-ramp for txs. |
| **Constant** | Immutable module-level value (inlined into bytecode). | `static final` field | Module-level `CONSTANT = value` (convention) or `enum` | Error constants often use `EPascalCase`. Used for prices, error codes, etc. Cannot be mutated. |
| **Reference** (`&T`, `&mut T`) + Borrow Checker | Temporary "borrow" of a value (immutable or mutable) without transferring ownership. The borrow checker enforces no aliasing for mutable borrows, no use-after-move, etc. | Java/Python object references (everything is a reference, GC-managed, shared mutable state by default) | Same as Java | Move (like Rust) has explicit ownership + borrowing. No GC. `&mut` allows mutation under strict rules; prevents data races at compile time. References cannot be stored in structs (ephemeral). |
| **Move Semantics** / Ownership | By default, values are *moved* (ownership transferred) on assignment or function call. Original variable becomes invalid. `copy` ability allows duplication. | Assignment copies the reference (shared); primitives are copied by value. Ownership is implicit via GC. | Same as Java (references) | Move: unique ownership by default (linear types). Prevents "use after free" or double-spend statically. Use `let mut` for rebindable vars that will be mutated via refs. |
| **Generic** (Type Parameter) | `<T>` placeholder for any type (with optional constraints like `T: copy + store`). Enables reusable code. | Java generics `<T extends ...>` | Python `typing.Generic`, `TypeVar`, or just duck typing (no static enforcement) | Move generics are monomorphized at compile. Phantom generics for tagging without data. Constraints are ability-based. |
| **Phantom Type** | Type parameter marked `phantom` that does not appear in struct fields (or only in phantom positions). Used for tagging (e.g., currency type) without affecting abilities or storage. | No direct built-in (use marker types or PhantomData in Rust); generics with unused params | No direct; use TypeVar or generics for typing only | Critical for safe `Coin<USDC>` vs `Coin<DAI>` — the phantom prevents needing spurious abilities on the tag type. |
| **Assert / Abort** | `assert!(cond, code)` or `abort code` halts execution and reverts the entire transaction with a `u64` error code. No try/catch. | `assert` (throws `AssertionError`); exceptions with try/catch | `assert` statement (raises `AssertionError`); try/except | Move aborts are transactional reverts (all-or-nothing). Error codes + modules are returned. Use `#[error]` for messages (Move 2024+). No recovery in same tx. |
| **Object** (Sui) | A top-level persisted entity on-chain: a `struct` with `key` ability + `id: UID`. Has ownership (owned/shared/immutable), version, and can be transferred/shared/frozen. | Java object instance (in heap); or entity in ORM/DB | Python object; or row in DB / document | Sui objects are first-class: fast-path for owned, consensus for shared. Tracked by runtime with UID. Not just in-memory structs. |
| **Init Function** | Special `fun init(ctx: &mut TxContext)` run exactly once when the module/package is published (like a constructor for the module). | Static initializer block or `static {}` in class; or `main` / Spring `@PostConstruct` | Module-level code at import time; or `__init__.py` top-level | Cannot be called manually. Often used to create and share initial objects or mint admin caps. |
| **Witness** (incl. One-Time Witness) | A type (often a struct with no fields) used as proof of authorization or "I am the creator". OTW is a special singleton created only at publish time. | Token object or capability class; or marker interface | Token / context object; or singleton class | Used for safe patterns like coin creation (`TreasuryCap` witness). OTW guarantees single instantiation for publisher authority. |
| **Capability** | An object (often with `key`) that grants the holder privileged rights (e.g., admin cap). Passed by reference to gate functions. | Capability-based security (e.g., OS capabilities) or token objects; `Admin` class | Similar token / permission object passed around | Common pattern instead of `tx_context::sender()` checks. More secure and composable than address-based ACLs. |
| **Transaction** (PTB / Script) | A sequence of commands (Move calls, transfers, publishes) executed atomically. PTBs are the modern way to compose calls. | Method call or database transaction (JTA); or script | Function call or `with` transaction in DB libs | In Sui, all state changes happen in one atomic tx. PTBs allow complex client-side composition without custom Move scripts. |

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

**In-depth topic:** [References, Ownership & the Borrow Checker](basics/references.md) (how the checker works, rules, common errors, Sui-specific usage, 12+ best practices).

Topics implemented: [modules](basics/modules.md) • [structs](basics/structs.md) • [functions](basics/functions.md) • [objects](basics/objects.md) • [primitives / types / vector / option / string](basics/types.md) • [examples](examples.md) • [references/borrow checker](basics/references.md) • [generics & constraints](basics/generics.md) • [constants, asserts/aborts & error handling](basics/errors.md)
