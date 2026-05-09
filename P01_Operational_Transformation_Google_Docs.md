# P01 — Operational Transformation (OT)
## "How can two people type at the same time and not break each other's work?"

> **Series:** Learn to Think Like an Architect  
> **Problem:** Google Docs Real-Time Collaborative Editing  
> **Level:** Junior → Senior  
> **Core Concept:** Operational Transformation, Conflict Resolution, Distributed State

---

## The Story

It is 2006. Google engineers are building a document editor that can be shared and edited by multiple people simultaneously. The problem seems simple on the surface — just sync the document state between clients. But the moment two users type at the same time, something breaks. One user's text appears in the wrong place. Characters jump around. The document becomes corrupted.

The engineers go back to a 1989 academic paper by Ellis and Gibbs titled *"Concurrency Control in Groupware Systems."* Inside it: the foundational idea that would make Google Docs possible.

They called it **Operational Transformation**.

---

## The Core Problem — Without OT

Imagine Alice and Bob are editing the same document. The document starts as:

```
"abc"
```

- **Alice** wants to insert `"X"` at position 1 (between `a` and `b`) → she wants `"aXbc"`
- **Bob** wants to insert `"Y"` at position 2 (between `b` and `c`) → he wants `"abYc"`

They both send their operations to the server at the same millisecond.

```
Initial: "abc"
         0 1 2   ← character positions

Alice's op:  Insert('X', pos=1)   →  "aXbc"
Bob's op:    Insert('Y', pos=2)   →  "abYc"
```

If the server simply applies operations in the order it receives them, here is what happens:

```
Server receives Alice's op first:
  "abc" → Insert X at 1 → "aXbc"   ✓ Alice happy

Server then applies Bob's original op:
  "aXbc" → Insert Y at position 2 → "aXYbc"
  
  But Bob's position 2 on the original "abc" pointed to between b and c.
  On the new string "aXbc", position 2 is X — not b!
  Bob gets "aXYbc" instead of the intended "aXbYc".  ✗
```

**This is the collision problem.** Positions on the original document no longer mean the same thing after someone else has inserted or deleted characters. The server naively applied Bob's operation to a world that had already changed.

```
┌──────────────────────────────────────────────────────────────┐
│              Naive Approach — Without OT                     │
├──────────────────────────────────────────────────────────────┤
│  Initial:   a  b  c                                          │
│             0  1  2                                          │
│                                                              │
│  Alice:     Insert X at pos 1  (intended: between a and b)  │
│  Bob:       Insert Y at pos 2  (intended: between b and c)  │
│                                                              │
│  Server applies Alice first:   a  X  b  c                   │
│                                0  1  2  3                   │
│                                                              │
│  Server applies Bob at pos 2:  a  X  Y  b  c   ✗ WRONG     │
│  Bob's Y landed on position 2 which is now X, not b         │
└──────────────────────────────────────────────────────────────┘
```

---

## The OT Insight

> **Do not apply incoming operations blindly. Transform them first — so they reflect the world that has changed since they were created.**

Operational Transformation says: before applying Bob's operation to a document that has already been modified by Alice, **transform Bob's operation against Alice's operation** to shift its position accordingly.

Bob inserted Y at position 2 on the original `"abc"`.  
Alice inserted X at position 1 — that is *before* Bob's intended position.  
Every character at position ≥ 1 shifted one step to the right.  
So Bob's position must also shift: `2 + 1 = 3`.

**Apply Bob's transformed op:** Insert Y at position 3 on `"aXbc"` → `"aXbYc"` ✓

Alice gets her X between a and b. Bob gets his Y between b and c. Both intentions are preserved.

---

## Step-by-Step Example with Full Transformation

**Initial state:** `"abc"` (positions: 0=a, 1=b, 2=c)

```
Alice's op:   Insert('X', pos=1)   ← wants "aXbc"
Bob's op:     Insert('Y', pos=2)   ← wants "abYc"
```

### Case 1: Server applies Alice first, then Bob

```
Step 1 — Apply Alice's op directly (it arrived first, no transformation needed):
  "abc" → Insert X at pos 1 → "aXbc"
  New positions: 0=a, 1=X, 2=b, 3=c

Step 2 — Transform Bob's op against Alice's op:
  Bob's original position = 2
  Alice inserted at position 1, which is ≤ Bob's position
  → Shift Bob's position right by 1: new position = 3

Step 3 — Apply Bob's transformed op:
  "aXbc" → Insert Y at pos 3 → "aXbYc"  ✓
```

**Result:** `"aXbYc"` — Alice's X sits between a and b. Bob's Y sits between b and c. ✓

---

### Case 2: Server applies Bob first, then Alice

```
Step 1 — Apply Bob's op directly (it arrived first now):
  "abc" → Insert Y at pos 2 → "abYc"
  New positions: 0=a, 1=b, 2=Y, 3=c

Step 2 — Transform Alice's op against Bob's op:
  Alice's original position = 1
  Bob inserted at position 2, which is > Alice's position
  → No shift needed: Alice's position stays at 1

Step 3 — Apply Alice's transformed op:
  "abYc" → Insert X at pos 1 → "aXbYc"  ✓
```

**Result:** `"aXbYc"` — same as Case 1. ✓

This is the key property of OT: **convergence**. No matter what order the server applies operations, all clients reach the same final state.

---

## The Transformation Rules

### Insert vs Insert

```
transform(insert_A(posA, charA), insert_B(posB, charB)):

  if posA < posB:
      return insert_A(posA, charA)        // A is before B — no shift needed

  else if posA > posB:
      return insert_A(posA + 1, charA)    // B shifted everything right of B by 1

  else:  // posA == posB — tie-break by client ID
      if clientA.id < clientB.id:
          return insert_A(posA, charA)    // A wins, stays at same position
      else:
          return insert_A(posA + 1, charA) // A loses, shifts right
```

### Insert vs Delete

```
transform(insert_A(posA, char), delete_B(posB)):

  if posA <= posB:
      return insert_A(posA, char)         // Delete is after insert — no shift

  else:
      return insert_A(posA - 1, char)     // Delete shifted everything left
```

### Delete vs Insert

```
transform(delete_A(posA), insert_B(posB)):

  if posA < posB:
      return delete_A(posA)               // Insert is after delete — no shift

  else:
      return delete_A(posA + 1)           // Insert shifted target right
```

### Delete vs Delete

```
transform(delete_A(posA), delete_B(posB)):

  if posA < posB:
      return delete_A(posA)               // No overlap

  else if posA > posB:
      return delete_A(posA - 1)           // B deleted a char before A's target — shift left

  else:
      return null                         // Both deleted the same character — A is a no-op
```

---

## Visual Analogy — The Line of People

Picture a line of people, each standing on a numbered tile:

```
Tile:    0    1    2    3
Person:  a    b    c
```

- Alice wants to squeeze into tile 1 (between person 0 and person 1).
- Bob wants to squeeze into tile 2 (between person 1 and person 2).

If they both jump at the same time without coordination, they collide. But OT gives Bob a rule:

> **"If someone squeezes in before your tile number, step one tile to the right before you jump."**

Alice squeezes into tile 1. All people at tile 1 and above shift right by one. Bob's target tile (2) was at or after Alice's insertion point (1), so Bob shifts his target: `2 → 3`. Bob then squeezes into tile 3. Everyone ends up in the right place.

That is OT.

---

## Code Example — JavaScript (Conceptual)

```javascript
// Operation types
// insert: { type: 'insert', position: number, text: string, clientId: string }
// delete: { type: 'delete', position: number, length: number }

function transform(op, againstOp) {
    if (op.type === 'insert' && againstOp.type === 'insert') {
        if (op.position < againstOp.position) {
            return op; // no shift needed
        }
        if (op.position > againstOp.position) {
            return { ...op, position: op.position + againstOp.text.length };
        }
        // same position — tie-break by clientId
        if (op.clientId < againstOp.clientId) {
            return op;
        }
        return { ...op, position: op.position + againstOp.text.length };
    }

    if (op.type === 'insert' && againstOp.type === 'delete') {
        if (op.position <= againstOp.position) {
            return op;
        }
        return { ...op, position: op.position - againstOp.length };
    }

    if (op.type === 'delete' && againstOp.type === 'insert') {
        if (op.position < againstOp.position) {
            return op;
        }
        return { ...op, position: op.position + againstOp.text.length };
    }

    if (op.type === 'delete' && againstOp.type === 'delete') {
        if (op.position < againstOp.position) {
            return op;
        }
        if (op.position > againstOp.position) {
            return { ...op, position: op.position - againstOp.length };
        }
        return null; // both deleted same character — this op is a no-op
    }
}
```

> **Note:** Production OT libraries handle rich text, multi-character strings, and complex edge cases. Use **ShareDB** or **ot.js** in real projects — but understand the theory so you can debug them.

---

## The Server Protocol — How Google Docs Uses OT

```
┌─────────────────────────────────────────────────────────────────────┐
│                    OT Server Protocol                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Client A (at v5)                    Client B (at v5)              │
│       │                                    │                        │
│       │  op_A + basedOnVersion=5           │  op_B + basedOnVersion=5 │
│       │──────────────────────────→         │──────────────────────→  │
│                                   SERVER                            │
│                               (currently at v7)                     │
│                                                                     │
│  1. Receive op_A (basedOn=5)                                        │
│  2. Server is at v7 — transform op_A against v6, v7                 │
│  3. Apply op_A′ → server moves to v8                               │
│  4. Broadcast op_A′ to all other clients                            │
│                                                                     │
│  Client A ←── ACK: your op is now v8                               │
│  Client B ←── op_A′ (transformed op to apply on top of their v7)   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

Key properties:
- The server is the **single source of truth** and the **serialization point** — all operations get a version number and are linearized here
- Clients apply their own operations **optimistically** (immediately, without waiting for the server) for zero-lag feel, then reconcile when the server ACK arrives
- Every operation is stored in an **op log**: `(docId, version, userId, opType, position, content, timestamp)` — the document text is always reconstructable by replaying the log from a snapshot

---

## Per-User Undo — The Hard Part

This is the problem most developers forget about when designing collaborative editors.

**Naive undo:** pop the last operation off a stack → broken, because the last operation might be someone else's.

**OT undo:** each user maintains their own undo stack containing only *their* operations. When a user hits Ctrl+Z:

```
1. Take the user's last operation from their personal undo stack.
2. Compute its inverse:
     Insert('X', pos=3) → inverse is Delete(pos=3, len=1)
     Delete(pos=3, len=1) → inverse is Insert('X', pos=3)
3. Transform the inverse against every operation that happened 
   AFTER the original operation (from all users).
4. Apply the transformed inverse.
```

**Walk-through:**

```
History log:
  v1: Alice inserts "hello" at pos 0  → doc: "hello"
  v2: Bob   inserts " world" at pos 5 → doc: "hello world"
  v3: Alice hits Ctrl+Z

Step 1: Alice's last op was Insert("hello", pos=0)
Step 2: Inverse = Delete(pos=0, len=5)
Step 3: Transform Delete(0,5) against Insert(" world", pos=5)
        → Delete is at pos 0-4, Insert was at pos 5
        → No positional overlap — Delete stays at pos 0, len=5
Step 4: Apply Delete(pos=0, len=5) to "hello world" → " world"  ✓

Bob's " world" is untouched. Alice's "hello" is gone. Intentions preserved.
```

---

## Mini Exercise — Test Yourself

**Start with:** `"123"`  
**User A:** Insert `"X"` at position 1  
**User B:** Delete the character at position 2 (the original `"2"`)

Work through both orderings. Do both converge to the same result?

<details>
<summary>Answer</summary>

**Case 1: Apply A first, then B**
- Apply A: `"123"` → Insert X at 1 → `"1X23"`
- Transform B's delete(pos=2) against A's insert(pos=1):
  - A inserted at pos 1, which is ≤ B's target (pos 2)
  - Shift B's position right by 1: new pos = 3
- Apply transformed B: Delete at pos 3 on `"1X23"` → delete `"2"` → `"1X3"` ✓

**Case 2: Apply B first, then A**
- Apply B: `"123"` → Delete at pos 2 → `"13"`
- Transform A's insert(pos=1) against B's delete(pos=2):
  - B deleted at pos 2, which is > A's position (1)
  - A's position stays at 1
- Apply transformed A: Insert X at pos 1 on `"13"` → `"1X3"` ✓

**Both converge to `"1X3"`** — OT works.
</details>

---

## OT vs CRDT — When to Use Which

```
┌─────────────────────┬──────────────────────────┬──────────────────────────┐
│                     │ OT                       │ CRDT                     │
├─────────────────────┼──────────────────────────┼──────────────────────────┤
│ Topology            │ Centralized server       │ Peer-to-peer / offline   │
│ Who uses it         │ Google Docs, Quip        │ Figma, Automerge, Yjs    │
│ Undo complexity     │ Simpler (server has      │ Complex (no total order  │
│                     │ total operation order)   │ of operations)           │
│ Implementation      │ Medium complexity        │ Higher complexity        │
│ Network requirement │ Needs server to          │ Works fully offline,     │
│                     │ serialize ops            │ syncs on reconnect       │
│ Scale               │ Server is a bottleneck   │ No central bottleneck    │
└─────────────────────┴──────────────────────────┴──────────────────────────┘
```

**For the Google Docs problem with per-user undo:** OT is the right choice. You have a central server, you need simple undo semantics, and the total operation order the server provides makes transformation deterministic.

---

## What You Need to Remember

| Concept | Takeaway |
|---------|----------|
| **Why OT?** | To merge concurrent edits without conflicts, preserving each user's original intention |
| **How?** | Before applying an op, transform it against all concurrent ops that have already been applied |
| **The rule** | If a remote op happened before your op's position → shift your position right. If after → no shift |
| **Convergence** | No matter what order ops are applied, all clients reach the same final state |
| **Undo** | An inverse op transformed against all subsequent history — not a global stack pop |
| **Do I implement OT?** | No — use **ShareDB** or **ot.js** — but understand the theory so you can debug and extend it |

---

## The Architect's Takeaway

OT feels like magic until you draw a timeline. Then it reveals itself as a clean, elegant idea:

> **Every operation is an instruction written for a world that no longer exists by the time it arrives. Your job is to translate it — not ignore it.**

This principle generalises far beyond text editors. Any time you have concurrent state changes across distributed nodes — shopping cart updates, collaborative design tools, multiplayer game state — you face the same fundamental question: *how do you merge intentions that were formed on different views of reality?*

OT is one answer. Understanding it deeply makes you a better architect regardless of whether you ever build a document editor.

---

## Libraries to Know

| Library | Language | Used By |
|---------|----------|---------|
| **ShareDB** | Node.js | Used by many real-time collab tools |
| **ot.js** | JavaScript | Lightweight OT client/server |
| **Yjs** | JavaScript | CRDT-based (alternative to OT) |
| **Automerge** | JavaScript / Rust | CRDT-based, good for offline-first |

---

*P01 · Learn to Think Like an Architect · [YouTube](https://www.youtube.com/@CodeWithSunchitDudeja) · [Instagram](https://www.instagram.com/sunchitdudeja/)*
