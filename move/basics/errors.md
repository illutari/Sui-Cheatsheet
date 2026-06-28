# Constants, Asserts/Aborts, and Error Handling

Move has no exception catching. A transaction either succeeds completely or aborts (reverting all changes). `abort` and `assert!` are the primary ways to fail a transaction early with a `u64` error code.

Good error handling makes debugging, user experience, and composability much better on Sui.

## Constants

Constants are module-level immutable values. They are inlined into bytecode.

```move
const MAX_SUPPLY: u64 = 1_000_000;
const ENotAuthorized: u64 = 0;  // error constant
```

### Naming

- Regular constants: `ALL_CAPS_WITH_UNDERSCORES` (compiler enforces starting with uppercase).
- Error constants: `EPascalCase` (e.g. `ENotFound`, `EInvalidAmount`).

See the [code quality checklist](https://move-book.com/guides/code-quality-checklist) for conventions.

### The Config Pattern (for sharing across modules)

Constants are private to their module. To expose config values:

```move
module my_pkg::config;

const ITEM_PRICE: u64 = 100;
const TAX_RATE: u64 = 10;

public fun item_price(): u64 { ITEM_PRICE }
public fun tax_rate(): u64 { TAX_RATE }
```

Other modules import and call the getters. On upgrade, only the config module needs changing.

## abort

```move
if (!has_access) {
    abort 1
}

abort ENotAuthorized;  // with constant
```

`abort` (and `abort <code>`) is an expression that can appear anywhere an expression of any type is expected (though it never produces a value).

It immediately halts the current function and aborts the *entire transaction*.

## assert!

Convenience macro:

```move
assert!(condition, error_code);
```

Expands to:

```move
if (!condition) { abort error_code }
```

The error code can be a literal or (preferably) a constant.

In Move 2024+, you can use `#[error]` annotated constants for richer messages (see below).

## Error Constants + Best Practices

Use descriptive constants instead of magic numbers.

### Rule 1: Check before calling (handle aborts in the caller module)

```move
// Bad: hard to know which call failed
public fun do_something() {
    module_b::get_field(1); // might abort 0
    // ...
    module_b::get_field(2); // might abort 0
}
```

Better:

```move
const ENoField1: u64 = 0;
const ENoField2: u64 = 1;

public fun do_something() {
    assert!(module_b::has_field(1), ENoField1);
    let f1 = module_b::get_field(1);
    // ...
    assert!(module_b::has_field(2), ENoField2);
    let f2 = module_b::get_field(2);
}
```

### Rule 2: Use different codes per scenario

Different codes let the caller (or frontend) map `0` → "Field 1 missing", `1` → "Field 2 missing", etc.

### Rule 3: Prefer returning `bool` from libraries; `assert!` in the app layer

Public library functions should often return `bool` (or `Option`) instead of asserting internally. Let the *caller module* decide the abort code and message.

```move
// In library
public fun is_authorized(...): bool { ... }

// In app
public fun do_a() {
    assert!(my_lib::is_authorized(), ENotAuthorized);
}
```

You can still have a private `assert_is_authorized()` helper inside the app module for DRY.

## #[error] Constants (Move 2024+)

Annotate error constants with `#[error]` to associate a human-readable message (as `vector<u8>` or string literal in some contexts). The VM/explorer can surface better messages.

```move
#[error]
const ENotEnoughBalance: vector<u8> = b"Insufficient balance for the operation";

assert!(balance >= amount, ENotEnoughBalance);
```

This improves UX significantly when the error bubbles up to wallets/explorers.

## Common Patterns in Sui

- Prefix all errors in a module with `E`.
- Group related errors (e.g. `EInvalidAmount`, `EAmountTooSmall`).
- Document each error constant with `///`.
- In tests: use `#[expected_failure(abort_code = EMyError)]` (or the constant directly in recent versions).
- For shared objects or privileged ops: combine with capabilities + clear error codes.

## Testing Aborts

```move
#[test]
#[expected_failure(abort_code = ENotAuthorized)]
fun test_unauthorized() {
    // ...
}
```

You can also use `abort_code = <module>::<CONST>`.

## Best Practices Summary (from Move Book guides)

1. Always handle possible abort scenarios in the *calling* module when possible.
2. Use distinct, descriptive error codes for different failure modes.
3. Expose "check" functions (`has_xxx`, `is_xxx`) that return `bool` from libraries.
4. Keep `assert!` (with app-specific error codes) in the application layer, not deep in shared libraries.
5. Use `#[error]` attributes (Move 2024) for user-friendly messages.
6. Never hard-code magic numbers in `assert!` or `abort` in production code.
7. Make error constants public if other modules might want to match on them (rare).
8. In PTBs / off-chain code, parse the module address + abort code to give good UX.

## Further Reading

- [Aborting Execution (Move Book)](https://move-book.com/move-basics/assert-and-abort.md)
- [Constants (Move Book)](https://move-book.com/move-basics/constants.md)
- [Better Error Handling (guide)](https://move-book.com/guides/better-error-handling.md)
- [Abort and Assert (Reference)](https://move-book.com/reference/abort-and-assert.md)
- [Code Quality Checklist - Error Constants](https://move-book.com/guides/code-quality-checklist)
- [Unit Testing - expected_failure](https://move-book.com/reference/unit-testing/)

---

*Previous: [Generics and Constraints](generics.md)*
*Related: [Move Summary](../move_summary.md)*
*See the [Better Error Handling guide](https://move-book.com/guides/better-error-handling.md) for more patterns*
