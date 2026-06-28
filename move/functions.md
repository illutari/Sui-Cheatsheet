# Move Functions

Functions contain the executable logic of your modules.

## Function Declaration

```move
// Basic private function
fun add(a: u64, b: u64): u64 {
    a + b
}

// Public API
public fun add_public(a: u64, b: u64): u64 {
    add(a, b)
}

// Entry point callable from transactions / PTBs
public entry fun increment_and_log(c: &mut Counter, ctx: &TxContext) {
    c.value = c.value + 1;
    // event::emit(...)
}
```

## Visibility Modifiers

| Syntax                  | Callable from other modules? | Can be called as transaction entry? | Typical Use |
|-------------------------|------------------------------|-------------------------------------|-------------|
| `fun`                   | No                           | No                                  | Internal helpers |
| `public fun`            | Yes                          | No (unless also entry)              | Library functions |
| `public(package) fun`   | Same package only            | No                                  | Package-internal API (Move 2024) |
| `entry fun`             | No                           | Yes                                 | Tx entry (private to module) |
| `public entry fun`      | Yes                          | Yes                                 | Most common for user-callable logic |

## Multiple Return Values

```move
public fun split(n: u64): (u64, u64) {
    (n / 2, n - n / 2)
}

let (half, rest) = split(10);
```

## References (`&` and `&mut`)

```move
// Immutable borrow (read-only)
public fun get_value(c: &Counter): u64 { c.value }

// Mutable borrow (can modify)
public fun set_value(c: &mut Counter, v: u64) {
    c.value = v;
}
```

The Move borrow checker prevents aliasing issues at compile time.

## Generics

```move
public fun wrap<T>(value: T): Wrapper<T> {
    Wrapper { value }
}

public struct Wrapper<T> has key, store {
    id: UID,
    value: T,
}
```

## Assert / Abort (error handling)

```move
const EInvalid: u64 = 1;

public fun do_something(v: u64) {
    assert!(v > 0, EInvalid);
    // or
    if (v == 0) abort EInvalid;
}
```

`assert!` is preferred; both stop execution and revert the transaction.

## Module Initializer `init`

Special function that runs exactly once when the package is published.

```move
fun init(ctx: &mut TxContext) {
    // create and share/transfer objects, set up state
    let cap = AdminCap { id: object::new(ctx) };
    transfer::transfer(cap, tx_context::sender(ctx));
}
```

- Must be named `init`.
- Signature usually `fun init(ctx: &mut TxContext)`
- Cannot be called manually.

## Best Practices

- Prefer `public entry` for functions meant to be called by users.
- Use `&mut` only when you need to mutate.
- Keep functions small and focused.
- Document public functions with `///` comments (shows in explorer / IDE).
- Use `public(package)` to hide implementation details within a package.

## Example

```move
public entry fun transfer_coin(coin: Coin<SUI>, recipient: address) {
    transfer::public_transfer(coin, recipient);
}
```

---

*Previous: [Structs](structs.md)*
*Next: [Objects](objects.md)*
