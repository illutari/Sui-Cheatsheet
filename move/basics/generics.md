# Generics and Constraints

Generics (also called type parameters) allow you to write reusable code that works over many types. They are the foundation of the standard library (`vector<T>`, `Option<T>`, etc.) and many Sui patterns (generic coins, wrappers, tables).

## Generic Syntax

Type parameters go in angle brackets after the name.

### Generic Structs

```move
/// A container that can hold any type T.
public struct Container<T> has key, store {
    value: T,
}
```

### Generic Functions

```move
/// Identity function - works for any T.
public fun id<T>(x: T): T { x }
```

You can have multiple: `<T, U, V>`

Instantiation is often inferred by the compiler:

```move
let c: Container<u64> = Container { value: 42 };
let c2 = Container { value: 42u64 }; // infers T = u64
```

## Phantom Type Parameters (Important for Sui)

When a type parameter is **not used** in the struct body (or only in phantom positions), declare it `phantom` so it doesn't affect ability derivation.

Classic example: different currencies for a Coin type.

```move
public struct Coin<phantom Currency> has key, store {
    value: u64,
}

// Different currencies don't need to have abilities themselves
public struct USD {}
public struct EUR {}

public fun mint_usd(value: u64): Coin<USD> { Coin { value } }
```

Without `phantom`, you'd be forced to give `USD` the `store` ability (or similar), which is unnecessary and can be a footgun.

Rules (enforced by compiler):
- Phantom params must only appear in phantom positions (i.e., as args to other phantom params, or not at all in the struct).
- The compiler warns if a param *could* be phantom but isn't marked as such.

Phantom params can still have ability constraints (see below), which *are* checked at instantiation.

## Constraints

Without constraints, the type system assumes the "worst" (a type with no abilities). Constraints let you require abilities on the type argument.

Syntax: `T: ability1 + ability2`

```move
public fun copy_and_drop<T: copy + drop>(x: T) {
    let _a = x; // copy
    let _b = x; // drop original too
}

public fun store_in_object<T: store>(val: T): Wrapper<T> { ... }
```

Common constraints in practice:
- `copy + drop` for small pure values
- `store` for things you want to put inside objects or dynamic fields
- `key` (rarely on generics, since key usually implies the whole thing is an object)

Constraints are checked at call sites / instantiation sites.

See also the abilities page for how abilities are derived on generic structs (`Cup<T>` has `copy` only if `T` has `copy`, etc.).

## Limitations

### Recursive Structs

You cannot have a struct that (directly or indirectly) contains a field of its own type, even with different type args. This prevents infinite size types.

Invalid examples are rejected at compile time.

### Type-Level Recursion Limits

Move forbids certain recursive generic function calls that could generate infinitely many distinct types at compile time (conservative static check, not based on runtime values).

Allowed (finite):
```move
fun foo<T>() { foo<T>(); } // or with phantom wrappers
```

Not allowed (can generate infinite types):
```move
fun foo<T>() { foo<Wrapper<T>>(); }
```

This is mostly an implementation limitation to keep the compiler/VM simple.

## Best Practices

- Use generics for library-like reusable code (collections, wrappers, abstract interfaces via capabilities).
- Prefer `phantom` for "tag" or "witness" type parameters (currency, permissions, etc.) to avoid spurious ability requirements.
- Add the minimal constraints you need (`store` is very common for Sui objects/dynamic fields).
- Combine with abilities on the generic struct itself: `public struct Box<T: store> has key, store { ... }`
- For conditional logic, use traits via ability constraints rather than runtime checks when possible.
- Document the expected constraints and phantom nature clearly.
- In Sui: generic over `phantom Witness` or `phantom Currency` is extremely common (see `sui::coin::Coin<C>`).

## Examples

### Generic Wrapper with Constraint

```move
public struct Box<T: store> has key, store {
    id: UID,
    inner: T,
}

public fun wrap<T: store>(inner: T, ctx: &mut TxContext): Box<T> {
    Box { id: object::new(ctx), inner }
}
```

### Using Phantom + Constraints Together

```move
public struct Ticket<phantom Event> has key, store { ... }

public struct Conference {} // no abilities needed

public fun buy_ticket(): Ticket<Conference> { ... }
```

## Further Reading

- [Generics (Move Book)](https://move-book.com/move-basics/generics.md)
- [Generics (Reference)](https://move-book.com/reference/generics.md)
- [Abilities + Conditional Abilities](https://move-book.com/reference/abilities.md) (especially conditional on generics)
- [Phantom types in Sui patterns](https://move-book.com/programmability/witness-pattern.md) and coin module

---

*Previous: [References & the Borrow Checker](references.md)*
*Next: [Constants, Asserts/Aborts, and Error Handling](errors.md)*
*Related: [Move Summary](../move_summary.md)*
