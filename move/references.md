# References, Ownership, and the Borrow Checker in Move

Move's type system is built around **ownership** and **borrowing**. This is what gives Move its strong safety guarantees without a garbage collector or runtime reference counting. The **borrow checker** (part of the type checker + bytecode verifier) statically enforces these rules at compile time.

This is heavily inspired by Rust but tailored for blockchain: emphasis on **linear types** (resources that must be explicitly used), no storing references in persistent state, and integration with Sui's object model.

## Ownership and Move Semantics (Recap)

Every value has exactly one **owner** (a scope or variable). When the owner scope ends, the value is dropped (unless it has `drop` ability).

- Defining a local: `let x = ...;` — the enclosing block/function owns `x`.
- Returning a value: transfers ownership to the caller.
- Passing by value to a function: **moves** ownership into the callee. After the call, the original variable is invalid.
- Blocks create sub-scopes.

```move
fun owner() {
    let a = 1; // owned by this function scope
    { let b = 2; } // b dropped here
} // a dropped here
```

Copyable types (those with `copy` ability: primitives like `u64`, `bool`, `address`, some small structs) are **automatically copied** instead of moved in many contexts. Resources (no `copy`) must be moved explicitly.

```move
let a = 10u64;
take(a); // copies (u64 has copy)
take(a); // still valid

let coin = get_coin(); // no copy
take_coin(coin); // moves
// take_coin(coin); // ERROR: use of moved value
```

Use `move` keyword to force a move even for copy types when you want to transfer ownership predictably.

## References: Borrowing Without Moving

References let you **lend** a value temporarily without giving up ownership. The lender retains ownership; the borrower gets a view (or edit capability).

Two kinds:
- `&T` — immutable reference (read-only).
- `&mut T` — mutable reference (can read + write).

Creating a reference is called **borrowing**.

```move
public fun is_valid(card: &Card): bool { card.uses > 0 }   // immutable borrow
public fun enter(card: &mut Card) { card.uses = card.uses - 1; } // mutable borrow
```

- The reference is valid only for the duration of the borrow (lexical scope + dataflow in the checker).
- After the function returns, the original owner can use the value again.
- You can have **multiple** `&` borrows at the same time.
- You can have **at most one** `&mut` at a time, and no overlapping `&` while a `&mut` is active.

## How the Borrow Checker Works

The borrow checker is a static analysis in the Move compiler (and re-verified on-chain in the Move VM bytecode verifier). It tracks:

1. **Ownership** of every value.
2. **Active borrows** (which references exist to which values/fields/paths).
3. **Mutability** of those borrows.

### Core Borrowing Rules (enforced everywhere, including in PTBs)

From the language + PTB execution model:

- **Mutable borrow (`&mut`)**: No other borrows (mutable or immutable) can be outstanding for the same value (or overlapping path). Prevents data races / dangling writes.
- **Immutable borrow (`&`)**: No mutable borrows can be outstanding. Multiple immutable borrows are fine (readers are safe).
- **Move (by value)**: No borrows (of any kind) can be outstanding. Moving a value while it is borrowed would invalidate references.
- **Copy**: Always allowed, even with outstanding borrows (for copyable types).

Additional rules:
- References cannot be stored in structs (or thus in objects/storage). They are **ephemeral** — exist only during a single transaction execution. (Big difference from Rust.)
- No references to references (`&&T` or `&mut &T` etc. are invalid).
- Field borrows: `&s.f` or `&mut s.f` are allowed and extend existing references when possible (same module for nested).
- You can reborrow: from `&mut T` you can get `&T` (via `freeze`) or narrower `&mut` to fields.

### Freeze Inference and Subtyping

`&mut T` is a **subtype** of `&T`. The compiler automatically inserts `freeze(e)` (converts `&mut T` → `&T`) where needed.

```move
fun takes_immut(x: &u64) { ... }
let mut x = 0;
takes_immut(&mut x); // OK — compiler freezes it
```

This appears in error messages as "is not a subtype of".

### Reading and Writing Constraints (tied to abilities)

- To **read** through a reference (`*ref` or `ref.field`): the underlying type `T` must have `copy` ability. (Prevents copying resources.)
- To **write** through a mutable reference (`*ref = new_val`): `T` must have `drop` ability. (Writing discards the old value.)
- This is why many asset structs have only `key, store` (no copy, no drop) — you cannot accidentally duplicate or destroy them via refs.

### What Errors the Checker Catches (Common Examples)

**Use after move:**
```move
let card = purchase();
recycle(card);           // moves ownership
// card.is_valid();      // ERROR: use of moved value
```

**Double mutable borrow / conflicting borrows:**
```move
let mut card = purchase();
let r1 = &mut card;
let r2 = &mut card;      // ERROR: cannot borrow `card` as mutable more than once
enter(r1);
```

**Borrow while moving:**
```move
let mut card = purchase();
let r = &card;
recycle(card);           // ERROR: cannot move `card` while borrowed
```

**Writing to non-droppable:**
```move
let mut ten = get_big_coin(); // assume Coin without drop
let r = &mut ten;
*r = other_coin;         // ERROR: would drop the original value
```

**Trying to store a reference:**
```move
public struct Bad has store {
    r: &u64,             // ERROR: references cannot be stored
}
```

**In PTBs (runtime + static enforcement similar):**
Arguments follow the same borrow/move/copy rules across the sequence of commands. You cannot move a result that is still borrowed by a later command. Move calls cannot return references.

The checker produces clear errors with source locations and explanations like "Invalid borrow", "value is moved", "overlapping mutable borrow", etc.

## Best Practices

1. **Prefer references over moves** when you only need to observe or mutate temporarily. This keeps ownership with the caller (especially important for capabilities and objects).

2. **Use `&` for reads, `&mut` only when mutation is required.** Many "getter" functions take `&Self`.

3. **Declare locals as `let mut`** when you (or a callee) will take a `&mut` to them.

4. **Keep borrow scopes small.** Borrow, use, let the borrow end before doing other operations on the value.

5. **For Sui objects:**
   - Entry functions on shared objects almost always take `&mut MyObject`.
   - Owned objects are often passed by value (moved) into transfer/consume functions.
   - Use `&Object` for read-only views or capability checks.
   - `TxContext` is usually `&mut TxContext` when you need to create objects/UIDs inside the function.

6. **Capabilities pattern:** Pass `&AdminCap` or `&mut AdminCap` to gate privileged operations without transferring the cap itself.

7. **Avoid fighting the checker with unnecessary moves.** If the compiler complains about a move, ask: "Do I really need to take ownership here, or can I borrow?"

8. **Use field reborrows for granular access:** `&mut obj.field` instead of borrowing the whole struct when possible.

9. **In tests/PTBs:** The `#[test]` or PTB execution will surface borrow violations the same way.

10. **Read the full error messages.** Move's errors are usually very precise about which value is borrowed, by whom, and why the operation is invalid.

11. **Sui-specific gotcha:** Shared objects have additional runtime rules (must stay shared, etc.). The borrow checker is only part of the story — the Sui verifier also checks storage invariants.

12. **For performance/clarity in PTBs:** Use `--preview` or dry-run to see how the runtime will borrow/move your arguments.

## Practical Example: Full Card Flow (Borrow Checker in Action)

See the metro card example in the Move Book (references chapter). It demonstrates:
- Purchase → returns owned `Card` (move out)
- `is_valid(&card)` — immutable borrow, card stays owned by caller
- `enter_metro(&mut card)` — mutable borrow + mutation
- `recycle(card)` — final move + consume (only allowed when no borrows active)

The test shows the checker allowing the sequence because borrows end before the next operation that would conflict.

## Further Reading & References

- [References (Move Book)](https://move-book.com/move-basics/references.md)
- [Ownership and Scope (Move Book)](https://move-book.com/move-basics/ownership-and-scope.md)
- [References (formal reference)](https://move-book.com/reference/primitive-types/references.md)
- [Abilities (copy/drop constraints)](https://move-book.com/move-basics/abilities-introduction.md)
- [PTB Borrowing Rules (Sui runtime enforcement)](https://docs.sui.io/develop/transactions/ptbs/prog-txn-blocks)
- Academic papers on Move's borrow checker and resource safety (linked from the book's appendix).

The borrow checker is one of the most powerful (and initially frustrating) features of Move. Once you internalize "everything has one owner, borrows are temporary loans with strict aliasing rules," most errors become intuitive.

---

*Previous: [Structs](structs.md) | [Functions](functions.md)*
*Related: [Move Summary](move_summary.md), [Objects](objects.md)*
