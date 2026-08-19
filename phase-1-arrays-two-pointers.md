# Phase 1 — Arrays, Two Pointers, Sliding Window & Prefix Sums
### Everything for Days 3–9, in plain English

This is the complete Phase 1 document. One file, nothing else needed.

After hashing, this is the highest-value phase in the roadmap. **Roughly half of the
array and string problems you get in a frontend interview are solved by one of four
skeletons in this document:** write pointer, opposite-end pointers, sliding window,
prefix sum. Memorize the four, learn the signal that picks one, and most "medium" array
problems stop being puzzles.

---

## How to use this document

Each trick gets its own Part, and every Part is laid out in the same boxes:

| Box | Question it answers |
|---|---|
| **What it is** | The idea, with a picture |
| **When to use it** | The literal words in the problem that trigger it |
| **The code** | The skeleton to memorize |
| **Watch it run** | Step by step on real numbers, so you can see it work |
| **Real problems** | LeetCode problems and the small change each one needs |
| **Mistakes** | What breaks, and how to spot it |

Want just the code? Jump to "The code" in any Part. Confused about *why*? Read "What it
is" and "Watch it run".

**Contents**

| Part | What's in it |
|---|---|
| 0 | Words used in this document (read first if any term is unfamiliar) |
| 1 | Why this phase exists |
| 2 | How to pick the right trick — the signal table |
| 3 | What a JavaScript array really is, and how to write clean loops |
| 4 | **TRICK 1 — Write pointer** (in place, return new length) |
| 5 | **TRICK 2 — Two pointers from opposite ends** (sorted, palindrome, ends) |
| 6 | **TRICK 2b — Sort first, then two pointers** (3Sum, 4Sum) |
| 7 | **TRICK 3 — Sliding window** (contiguous, longest/shortest) |
| 8 | **TRICK 4 — Prefix sums** (range sums, negatives allowed) |
| 9 | Bonus — Kadane (best chunk sum) |
| 10 | Bonus — use the array as its own lookup table |
| 11 | JavaScript traps (and what "V8" means) |
| 12 | Complexity of every pattern here |
| 13 | Where this actually shows up in frontend work |
| 14 | What to say out loud in an interview |
| 15 | The six checks before you say "done" |
| 16 | Your 7-day schedule |
| 17 | Practice — 32 problems |
| 18 | Worked answers to all 32 |
| 19 | One-page cheat sheet |

---

## Part 0 — Words used in this document

Read this once. Every confusing term in this file is defined here.
(The full course-wide dictionary is in `glossary.md`; this is the Phase 1 subset.)

**Array** — a numbered list of values. `[10, 20, 30]`. Positions are called **indexes**
and start at **0**, so `arr[0]` is 10.

**n** — how many items the input has. **k** — some number the problem gives you
("a window of size k").

**Pointer** — nothing to do with C pointers. It's just **a variable holding an index**.
"Move the left pointer" means `l++`.

| Variable | Short for | Means |
|---|---|---|
| `i`, `j` | index | ordinary loop counter |
| `l` / `r` | left / right | index of the left / right edge |
| `w` | write | index where the next kept value goes |
| `n` | number | the array's length |
| `k` | — | a size or limit the problem gives you |

**In place** — change the original array instead of building a new one.
**Mutate** — the same idea: to change something that already exists. `sort()` mutates.

**Big-O / O(n)** — how much slower the code gets as the input grows.

| Written | Plain meaning | For 1,000 items |
|---|---|---|
| O(1) | same cost regardless of size | 1 step |
| O(log n) | halve the problem each step | ~10 steps |
| O(n) | one pass | 1,000 steps |
| O(n log n) | what a good sort costs | ~10,000 steps |
| O(n²) | every item against every item | 1,000,000 steps |

**Auxiliary space** — extra memory *not counting the answer you were told to return*.
If the problem says "return an array of n results", that array doesn't count against you.

**Amortized** — averaged out over many operations. A single step may be expensive, but
the total across the whole run is still cheap.

**Contiguous** — side by side, no gaps.

**Subarray / substring** — a **contiguous** chunk. `[2,9,7]` inside `[4,2,9,7,1]`.
**Subsequence** — items in order but **skipping allowed**. `[4,9,1]` from the same array.

> ⚠️ **This one word decides your whole approach.** Subarray/substring → sliding window
> is possible. Subsequence → windows are useless, it's Dynamic Programming (Phase 10).
> Look for this word specifically when you read a problem.

**Window** — a continuous chunk of the array, marked by two indexes `l` (left edge) and
`r` (right edge). Everything from `l` to `r` inclusive is "inside the window". That's the
entire definition. Full explanation with pictures in Part 7.1.

**Monotone** — "only moves one way, never flips back". A window's validity is monotone if
adding items can only push it toward invalid, never suddenly back to valid.

**Invariant** — a promise that is true at every step of the loop. Stating invariants out
loud earns real credit in interviews.

**Predicate** — a yes/no test, e.g. `x => x !== 0`.

**V8** — **the program that actually runs your JavaScript.** It's Google's JS engine, the
C++ inside Chrome, Edge, and Node. When this doc says "a V8 gotcha", read it as *"a
JavaScript quirk that a Python or Java candidate in the same interview wouldn't hit."*
That's all it means. Other engines (Safari's JavaScriptCore, Firefox's SpiderMonkey)
behave the same for everything here.

**Hash map** — a lookup table: give it a key, get a value back instantly. In JS that's
`Map`. **Set** — the same thing but keys only, for "have I seen this before?".

**Dry run** — tracing your code on paper, writing down every variable at every step,
*before* clicking Run.

**LC 15** — "LeetCode problem number 15", so you can search for it.

**Code shorthand you'll see:**

| You see | It means |
|---|---|
| `w++` | use `w`, then add 1 |
| `arr[w++] = x` | write `x` at index `w`, **then** move `w` forward |
| `r - l + 1` | the **length** of the window from `l` to `r` (`+1` because both ends count) |
| `[a[i], a[j]] = [a[j], a[i]]` | swap two elements |
| `(a, b) => a - b` | "sort numerically" — always required for numbers |
| `Infinity` / `-Infinity` | starting values for "best minimum" / "best maximum" |
| `map.get(x) \|\| 0` | "the count so far, or 0 if never seen" |

**Why `r - l + 1` and not `r - l`:** if `l = 2` and `r = 2`, the window holds exactly one
element. `r - l` gives 0 (wrong); `r - l + 1` gives 1 (right). Check it that way every
time.

---

## Part 1 — Why this phase exists

Phase 0 taught you to *measure* code. This phase teaches you to *hit* a target complexity
on purpose.

Almost every problem here has the same shape:

> The obvious solution checks **every pair** or **every possible chunk** — that's O(n²)
> or worse. There is some structure hidden in the problem that lets you avoid
> re-checking what you already looked at. Find it, and you drop to O(n).

Only a few kinds of structure make that possible, and each one maps to exactly one trick:

| Structure hiding in the problem | Trick it unlocks |
|---|---|
| The array is **sorted** → comparing the two ends tells you which end to throw away | Two pointers, opposite ends |
| The answer is a **shorter version of the same array** → a write index can trail a read index | Write pointer |
| The answer is a **continuous chunk**, and adding items only pushes it one direction | Sliding window |
| You need **sums over ranges** → a range sum is the difference of two running totals | Prefix sum |
| Values are **numbers 1..n** → the array can be its own lookup table | Index-as-hash |

When you stall in an interview, **walk that table out loud**. It isn't filler — it's the
actual search you're performing, and interviewers score it.

**What "done" looks like:** you read an unfamiliar array problem, and within 60 seconds
you name the trick and the target complexity — *before* writing any code — then write the
skeleton from memory without re-deriving the boundaries.

---

## Part 2 — How to pick the right trick

The single most useful page in the document. When stuck, read down the left column out
loud until something matches.

| The problem says… | It almost always means |
|---|---|
| "sorted array" + "find a pair / triplet summing to X" | **Two pointers from the ends** (sort first if it isn't sorted) |
| "palindrome", "reverse in place", "swap the ends" | **Two pointers from the ends** |
| "in place", "return the new length", "O(1) extra space" | **Write pointer** |
| "contiguous subarray" or "substring" + "longest / shortest / max / min" | **Sliding window, variable size** |
| "subarray of size k", "every window of k", "average of every k" | **Sliding window, fixed size** |
| "at most k distinct / at most k replacements / at most k zeros" | **Variable window**, shrink while the k-rule is broken |
| "**exactly** k distinct" | `atMost(k) − atMost(k−1)` — run the window twice, subtract |
| "**count** the subarrays where…" | Window, add `r − l + 1` each step |
| "sum of a range", "many range queries", "subarray sums to k" | **Prefix sum** (+ hash map if you must *count* them) |
| "subarray sum divisible by k" | Prefix sum **mod k** + hash map |
| "equal number of 0s and 1s" | Turn every 0 into −1 → "prefix sum revisits a value" |
| "product of all except self", "without division" | Prefix **and** suffix products |
| "maximum sum subarray", "best profit", "max product subarray" | **Kadane** (running value) |
| "numbers are in the range 1..n" + "O(1) space" | **Index-as-hash** or cyclic sort |
| "merge two sorted arrays in place" | Two pointers filling **from the back** |
| "only 0s, 1s and 2s, sort in one pass" | Dutch national flag (three pointers) |
| Not sorted, need pairs summing to a target, and you need **indexes** | **Hash map** — that's Phase 2, not this phase |

### The one distinction people get wrong

Both tricks use two pointers, so beginners blur them. They are not the same thing:

|  | Two pointers, opposite ends | Sliding window |
|---|---|---|
| Where they start | `l` at the left edge, `r` at the right edge | both at the left |
| Which way they move | **toward each other** | **the same direction** (rightward) |
| What it needs | the array to be **sorted** | the answer to be **contiguous** |
| What it finds | a pair / a boundary | a chunk |
| Loop shape | `while (l < r)` | `for (r…) { while (bad) l++ }` |

Candidates who blur these cannot explain why their loop terminates, and that's exactly
what gets asked.

---

## Part 3 — What a JavaScript array really is (and how to write clean loops)

Boring section. **Most bugs live here.** Ten minutes now saves you an interview later.

### 3.1 A JS array is not a row of numbers in memory

In C or Java, an array is a solid block of memory. In JavaScript it's an **object** whose
keys happen to look like numbers, which V8 (see Part 0) then optimizes into something
array-shaped *when it can*.

You will not be quizzed on the internals. Three consequences do matter.

### 3.2 Holes — the part that actually bites

A **hole** is a slot that doesn't exist at all, which is different from a slot holding
`undefined`.

```js
const a = new Array(3);          // [ <3 empty items> ] — three HOLES
a[0];                            // undefined — but via a slow path
0 in a;                          // false   ← the tell: the slot isn't there

const b = new Array(3).fill(0);              // [0,0,0]  ✅
const c = Array.from({ length: 3 }, () => 0); // [0,0,0]  ✅

const d = [1, 2, 3];
delete d[1];                     // [1, <1 empty item>, 3] — punched a hole. Never do this.
d.length;                        // still 3
```

Holes also make array methods behave strangely:

```js
const holey = [1, , 3];
holey.map(x => x * 2);           // [2, <1 empty item>, 6]  — map SKIPS holes
holey.forEach(() => {});         // callback runs TWICE, not three times
holey.includes(undefined);       // true
holey.indexOf(undefined);        // -1     ← the two disagree
```

**Rule for this phase:** create result arrays with `new Array(n).fill(0)` or
`Array.from`. Never bare `new Array(n)`, never `delete` on an array.

### 3.3 The 2D array bug that catches everyone

```js
const grid = Array(3).fill([]);   // ❌ all three rows are the SAME array
grid[0].push(1);
grid;                             // [[1], [1], [1]]  😱

const ok = Array.from({ length: 3 }, () => []);                          // ✅
const dp = Array.from({ length: rows }, () => new Array(cols).fill(0));  // ✅
```

`fill` with an object fills every slot with **one reference**. `Array.from` with a
factory function calls it fresh per index. This bug shows up in DP tables (Phase 10),
grids (Phase 9), and real React code. Get it right by reflex.

### 3.4 Typed arrays — one sentence, if asked

`Int32Array`, `Float64Array`, `Uint8Array` are genuinely contiguous, fixed length,
zero-filled, and cannot have holes.

```js
const counts = new Uint8Array(26);   // zero-filled, hole-proof
```

Same Big-O, better constants, less memory. Worth one sentence when a problem says "the
array can hold 10 million integers". Not worth reaching for in a 20-minute interview
where clarity beats bytes.

### 3.5 The cost table you need for this phase

| Operation | Cost | What it means here |
|---|---|---|
| `arr[i]`, `arr.length` | O(1) | index math is always cheaper than slicing |
| `push` / `pop` | O(1) amortized | fine inside loops |
| `shift` / `unshift` / `splice` | **O(n)** | never inside a loop — use a write pointer |
| `slice`, `concat`, `[...arr]` | **O(n)** | never inside a window loop — this is how O(n) becomes O(n²) |
| `includes` / `indexOf` / `find` | **O(n)** | inside a loop → O(n²); use a `Set`/`Map` |
| `sort((a,b) => a-b)` | O(n log n) | **mutates the input** — say so out loud |
| `map` / `filter` / `reduce` | O(n) time and space | allocates; "in place" problems forbid them |

**Every pattern in this phase exists to avoid the O(n) rows of that table.** If your
"O(n) sliding window" calls `slice` to materialize the current window, it is actually
O(n²). That is the most common self-inflicted wound in this phase.

### 3.6 Which loop to use

```js
for (let i = 0; i < arr.length; i++) { }   // need the index, or need to jump → this
for (const x of arr) { }                    // values only → clean
arr.forEach((x, i) => { });                 // can't break out → avoid in algorithm code
for (const i in arr) { }                    // ❌ NEVER on arrays
```

Why `for...in` is always wrong on arrays:

```js
const a = [10, 20, 30];
for (const i in a) console.log(i + 1);   // "01", "11", "21" — the keys are STRINGS
```

Every pattern in this phase wants explicit index loops. **Pointers *are* indexes**;
hiding them behind an iterator makes the pattern impossible to write.

### 3.7 Empty and single-element inputs

```js
let r = arr.length - 1;        // -1 when the array is empty
while (l < r) { }              // 0 < -1 is false → loop skipped. Often correct by accident.
arr[arr.length - 1];           // undefined on empty — a silent wrong answer, not a crash
Math.max(...[]);               // -Infinity
[].reduce((a, b) => a + b);    // ❌ TypeError: Reduce of empty array with no initial value
```

`while (l < r)` loops are usually empty-safe for free. Anything that reads `arr[0]` or
`arr[n-1]` *before* the loop is not. Say the guard out loud — *"empty returns 0, a single
element returns itself"* — then write it.

### 3.8 The swap

```js
[a[i], a[j]] = [a[j], a[i]];              // idiomatic, O(1) — write this
const t = a[i]; a[i] = a[j]; a[j] = t;    // marginally faster in a hot loop
```

Use the destructuring form. If someone asks about the temporary array it appears to
create, say V8's escape analysis normally removes it and it's O(1) either way.

### 3.9 Loop bounds you should recognize instantly

```js
for (let i = 0; i < n; i++)          // every element
for (let i = 1; i < n; i++)          // every element compared with i-1
for (let i = 0; i < n - 1; i++)      // every adjacent PAIR (i, i+1)
for (let i = n - 1; i >= 0; i--)     // right to left — suffix values, or writing safely
for (let i = 0; i + k <= n; i++)     // every window of size k, by start index
```

⚠️ Windows use `i + k <= n`, **not** `i + k < n`. The last valid window starts at `n - k`,
and `(n-k) + k <= n` is true. Getting this wrong silently drops the final window — a very
common bug, and the interviewer's test case usually catches it.

### 3.10 Compare backward, not forward

Prefer `arr[i-1]` with the loop starting at 1, over `arr[i+1]` with the loop ending at
`n-1`. Backward reads can never go out of bounds when the start index is right, and the
previous value is the one you've already validated.

---

## Part 4 — TRICK 1: The Write Pointer
### Use it when the problem says "in place" or "return the new length"

### 4.1 What it is

Two index variables walk left to right:

- **`r` (read)** visits every single element, one by one. It never skips.
- **`w` (write)** marks **the slot where the next keeper belongs**.

When `r` finds something worth keeping, copy it to slot `w` and push `w` forward. When
`r` finds junk, ignore it and `w` stays put. So `w` falls behind `r` by exactly the number
of items you've thrown away.

Removing the zeros from `[0, 1, 0, 3, 12]`:

```
                     r
[ 0 , 1 , 0 , 3 , 12 ]      r sees 0 → junk, skip. w stays at 0.
  ↑
  w

         r
[ 1 , 1 , 0 , 3 , 12 ]      r sees 1 → keeper. Write it at w. w moves to 1.
      ↑
      w
```

Because `w` is always **behind or equal to** `r`, writing at `w` can never destroy an
element you still need to read. That's why no second array is needed.

### 4.2 When to use it

The problem says any of:

- "Modify the array **in place**"
- "**Return the new length**"
- "Use only **O(1) extra space**"
- "The relative order of the remaining elements **must be preserved**"

### 4.3 The code

```js
function keepIf(arr, shouldKeep) {
  let w = 0;                          // how many keepers written so far
  for (let r = 0; r < arr.length; r++) {
    if (shouldKeep(arr[r])) {
      arr[w] = arr[r];
      w++;
    }
  }
  return w;                           // arr[0 .. w-1] is the answer
}
// Time  O(n) — every element read once, written at most once.
// Space O(1) — just two numbers.
```

**Say these two sentences out loud in the interview.** This is where the marks are:

1. *"`arr[0]` to `arr[w-1]` is always the correct answer for everything I've processed so far."*
2. *"`w` is always ≤ `r`, so I can never overwrite something I still need to read."*

Those two promises are the **invariants**. Stating them is the difference between "I've
seen this before" and "I understand this".

### 4.4 Watch it run — Move Zeroes (LC 283), `[0, 1, 0, 3, 12]`

```js
function moveZeroes(nums) {
  let w = 0;
  for (let r = 0; r < nums.length; r++) {
    if (nums[r] !== 0) nums[w++] = nums[r];   // pass 1: pack the non-zeros forward
  }
  while (w < nums.length) nums[w++] = 0;      // pass 2: fill the rest with zeros
}
// O(n) time, O(1) space.
```

| Step | `r` | `nums[r]` | Keep? | Action | Array after | `w` |
|---|---|---|---|---|---|---|
| start | – | – | – | – | `[0,1,0,3,12]` | 0 |
| 1 | 0 | 0 | no | skip | `[0,1,0,3,12]` | 0 |
| 2 | 1 | 1 | yes | `nums[0] = 1` | `[1,1,0,3,12]` | 1 |
| 3 | 2 | 0 | no | skip | `[1,1,0,3,12]` | 1 |
| 4 | 3 | 3 | yes | `nums[1] = 3` | `[1,3,0,3,12]` | 2 |
| 5 | 4 | 12 | yes | `nums[2] = 12` | `[1,3,12,3,12]` | 3 |
| fill | – | – | – | zeros from index 3 | `[1,3,12,0,0]` | 5 |

Notice step 5: the array looks **wrong** mid-flight (`12` appears twice). That's fine —
only `arr[0..w-1]` is promised to be correct. The junk on the right gets overwritten.

**The minimum-writes version**, worth mentioning if the interviewer asks about write count:

```js
function moveZeroesSwap(nums) {
  let w = 0;
  for (let r = 0; r < nums.length; r++) {
    if (nums[r] !== 0) {
      if (w !== r) [nums[w], nums[r]] = [nums[r], nums[w]];   // ⭐ skip pointless self-swaps
      w++;
    }
  }
}
// O(n) time, O(1) space. Writes only when a value actually has to move.
```

The `if (w !== r)` guard is a legitimate frontend point: when the "swap" is really
`insertBefore` on DOM nodes, every unnecessary move costs layout work. Skipping no-op
moves is the difference between reflowing every node and reflowing only the moved ones.

### 4.5 Real problems

| LeetCode | What changes |
|---|---|
| **27 Remove Element** | Keep if `nums[r] !== val` |
| **26 Remove Duplicates from Sorted Array** | Keep if `nums[r] !== nums[w-1]` — compare against the last thing you **kept**, not the last thing you **read** |
| **80 Remove Duplicates II (at most 2 copies)** | Keep if `w < 2 \|\| nums[r] !== nums[w-2]` |
| **283 Move Zeroes** | Keep if `nums[r] !== 0`, then fill the tail with 0 |

**Remove Duplicates from Sorted Array (LC 26):**

```js
function removeDuplicates(nums) {
  if (nums.length === 0) return 0;
  let w = 1;                                     // the first element is always a keeper
  for (let r = 1; r < nums.length; r++) {
    if (nums[r] !== nums[w - 1]) nums[w++] = nums[r];
  }
  return w;
}
// O(n) time, O(1) space.
```

Compare against `nums[w-1]` (the last **kept** value), not `nums[r-1]` (the last **read**
value). On sorted input both happen to work — but the moment the rule becomes "at most
two copies", only the kept-value form generalizes.

**Remove Duplicates II — at most k copies (LC 80).** The whole family in three lines:

```js
function removeDuplicatesAtMost(nums, k = 2) {
  let w = 0;
  for (const x of nums) {
    if (w < k || x !== nums[w - k]) nums[w++] = x;
  }
  return w;
}
// O(n) time, O(1) space.
```

**Why `nums[w - k]` works:** it's the value `k` slots back **in your output**. If the
incoming value differs from it, you've written fewer than `k` copies so far, so this one
is allowed. Writing the general `k` version instead of hardcoding 2 is a strong signal —
it shows you understood the invariant rather than memorizing the case.

**Remove Element, the "order doesn't matter" variant** — fewer writes when matches are rare:

```js
function removeElementSwap(nums, val) {
  let n = nums.length, i = 0;
  while (i < n) {
    if (nums[i] === val) nums[i] = nums[--n];   // pull from the end; DON'T advance i
    else i++;
  }
  return n;
}
// O(n) time, O(1) space.
```

⚠️ **Don't advance `i` after pulling.** The value you just pulled in from the end has
never been examined. This is the same trap as the `high` pointer in Sort Colors (Part
5.9): **whenever you move an unexamined element into the current slot, you must
re-examine that slot.**

### 4.6 Mistakes

| Mistake | What happens | Fix |
|---|---|---|
| Using `splice` to delete inside a loop | Every `splice` shifts all later elements → **O(n²)** | Use the write pointer |
| Using `filter` | Creates a new array → fails "in place, O(1) space" | Use the write pointer |
| Comparing to `nums[r-1]` instead of `nums[w-1]` | Works by luck, breaks on "at most 2 copies" | Compare against the last **kept** value |
| Forgetting the second pass in Move Zeroes | The tail still holds stale values | `while (w < n) nums[w++] = 0` |
| Advancing `i` after pulling from the end | Skips an unchecked value | Don't advance |

**The two wrong answers, so you recognize them:**

```js
// ❌ O(n²) — splice shifts everything after it, on every single removal
for (let i = 0; i < nums.length; i++) {
  if (nums[i] === val) { nums.splice(i, 1); i--; }
}

// ❌ O(n) extra space, and it doesn't modify in place, which is what was asked
return nums.filter(x => x !== val);
```

Both are what an untrained candidate writes. **Naming *why* they're wrong** — `splice` is
O(n) per call, `filter` allocates — is the entire point of Phase 0.

---
## Part 5 — TRICK 2: Two Pointers From Opposite Ends
### Use it when the array is sorted, or when the problem is about the two ends

### 5.1 What it is

Put one index at the far left, one at the far right. Look at both values. Decide which
side **cannot possibly** be part of the answer, and throw it away by moving its pointer
inward. Repeat until they meet.

```
[ 2 , 7 , 11 , 15 ]        target = 9
  ↑            ↑
  l            r           2 + 15 = 17. Too big. 15 is too large for ANY
                           partner → throw away the right side.

[ 2 , 7 , 11 , 15 ]
  ↑        ↑
  l        r               2 + 11 = 13. Still too big → move r again.

[ 2 , 7 , 11 , 15 ]
  ↑    ↑
  l    r                   2 + 7 = 9. Found it.
```

Each step throws away at least one element and never revisits it, so the loop runs at
most n times → **O(n) time, O(1) space**.

### 5.2 When to use it

- The array is **sorted** (or the problem lets you sort it)
- The problem is about **pairs or triplets summing to a target**
- The problem is about **symmetry**: palindrome, reverse, "swap the ends"
- The problem is about **two boundaries**: container walls, trapped water

### 5.3 The code

```js
let l = 0, r = arr.length - 1;
while (l < r) {
  // look at arr[l] and arr[r], decide which side is now useless
  if (someCondition) l++;
  else r--;
}
// O(n) time, O(1) space.
```

**`l < r` or `l <= r`?**

- Use **`l < r`** when you need two *different* positions — pairs, palindromes, areas.
  That's almost everything in Phase 1.
- Use `l <= r` only when the single middle element must itself be processed. That's
  binary search (Phase 3) — and Boats to Save People, problem 17 in the practice set.

Picking the wrong one gives an off-by-one on odd-length inputs. In a palindrome check
comparing the middle character to itself is harmless; in a pair-sum, returning `[i, i]`
is flat wrong.

**The question you must be able to answer for every problem here:**
*"Why is it safe to throw away a whole side?"* Each problem has a different one-sentence
answer, and that sentence is the actual interview question.

### 5.4 Watch it run — Container With Most Water (LC 11)

Walls of different heights. Pick two; the water they hold is
`width × height of the shorter wall`. Find the maximum.

```js
function maxArea(height) {
  let l = 0, r = height.length - 1, best = 0;
  while (l < r) {
    best = Math.max(best, (r - l) * Math.min(height[l], height[r]));
    if (height[l] < height[r]) l++;      // move the SHORTER side
    else r--;
  }
  return best;
}
// O(n) time, O(1) space.
```

On `[1, 8, 6, 2, 5, 4, 8, 3, 7]`:

| `l` | `r` | `h[l]` | `h[r]` | width | shorter wall | area | best | Move |
|---|---|---|---|---|---|---|---|---|
| 0 | 8 | 1 | 7 | 8 | 1 | 8 | 8 | `l++` (1 < 7) |
| 1 | 8 | 8 | 7 | 7 | 7 | **49** | 49 | `r--` (7 ≤ 8) |
| 1 | 7 | 8 | 3 | 6 | 3 | 18 | 49 | `r--` |
| 1 | 6 | 8 | 8 | 5 | 8 | 40 | 49 | `r--` |
| 1 | 5 | 8 | 4 | 4 | 4 | 16 | 49 | `r--` |
| 1 | 4 | 8 | 5 | 3 | 5 | 15 | 49 | `r--` |
| 1 | 3 | 8 | 2 | 2 | 2 | 4 | 49 | `r--` |
| 1 | 2 | 8 | 6 | 1 | 6 | 6 | 49 | `r--` → pointers meet, stop |

Answer: **49**.

**Why moving the shorter wall is safe** — memorize this, it *is* the problem:

> The area is capped by the **shorter** wall. Any other container that still uses that
> shorter wall must have a smaller width, and its height is still capped by that same
> shorter wall. So it can never beat the area I just recorded. Therefore the shorter wall
> is finished — discard it.

Candidates who move the taller wall (or both) pass some tests, fail others, and can't say
why.

### 5.5 Reverse in place, and the string caveat

```js
function reverse(arr) {
  let l = 0, r = arr.length - 1;
  while (l < r) {
    [arr[l], arr[r]] = [arr[r], arr[l]];
    l++; r--;
  }
  return arr;
}
// O(n) time, O(1) space. n/2 swaps is still O(n).
```

**Strings are immutable in JavaScript**, so there is no O(1)-space string reversal:

```js
[...str].reverse().join('')     // O(n) time AND O(n) space — the honest answer
```

Use `[...str]` rather than `str.split('')` so emoji (surrogate pairs) don't get torn in
half. Volunteer that this would be O(1) space in C++ — it's a real language difference,
not a weakness in your solution.

### 5.6 Valid Palindrome (LC 125)

Ignore punctuation and case:

```js
function isPalindrome(s) {
  const isAlnum = c => /[a-z0-9]/i.test(c);
  let l = 0, r = s.length - 1;
  while (l < r) {
    while (l < r && !isAlnum(s[l])) l++;      // skip junk on the left
    while (l < r && !isAlnum(s[r])) r--;      // skip junk on the right
    if (s[l].toLowerCase() !== s[r].toLowerCase()) return false;
    l++; r--;
  }
  return true;
}
// O(n) time, O(1) space.
```

Three things interviewers check:

1. The inner `while` loops **also** carry `l < r`. Without it, a string of pure
   punctuation runs a pointer off the end. Test `",."` and `"a."`.
2. The easy version — strip everything with `replace`, lowercase, compare to the reverse
   — is O(n) **space**. This version is O(1). Mention both, pick what they asked for.
3. **The nested `while` does NOT make this O(n²).** `l` and `r` only move toward each
   other, so their total movement across the whole run is at most n. Say this explicitly.

### 5.7 Two Sum II — sorted input (LC 167)

```js
function twoSum(nums, target) {
  let l = 0, r = nums.length - 1;
  while (l < r) {
    const sum = nums[l] + nums[r];
    if (sum === target) return [l + 1, r + 1];   // this problem wants 1-based indexes
    if (sum < target) l++;                        // need a bigger sum → only l can help
    else r--;                                     // need a smaller sum → only r can help
  }
  return [];
}
// O(n) time, O(1) space.
```

**Why discarding is safe:** if `nums[l] + nums[r]` is **below** the target, then `nums[l]`
paired with anything at or left of `r` is even smaller. So `nums[l]` can't be part of any
answer inside `[l, r]` — `l` is done forever. Same logic mirrored for `r`. Being able to
state that *is* the whole problem.

⚠️ **Important contrast.** The *unsorted* Two Sum (LC 1) is **not** this problem. Sorting
would cost O(n log n) **and** destroy the original indexes — which is exactly what LC 1
asks you to return. Unsorted Two Sum is a hash map problem (Phase 2): O(n) time, O(n)
space, no sorting. Say this trade-off out loud; it's a favourite follow-up.

### 5.8 Trapping Rain Water (LC 42) — the best demonstration in this phase

Tagged "hard", but worth learning here because it shows how to turn two prefix arrays
into O(1) space — the single most valuable move in this phase.

**The idea:** the water sitting above bar `i` is
`min(tallest bar to its left, tallest bar to its right) − height[i]`.

```
        █                     water is capped by the SHORTER of the two
    █░░░█ █                   tallest-so-far walls on either side
    █░█░█░█ █
  █ █ █ █ █ █
```

**Step 1 — the honest, easy version.** Precompute both maxima:

```js
function trapPrefix(h) {
  const n = h.length;
  if (n === 0) return 0;
  const leftMax = new Array(n), rightMax = new Array(n);

  leftMax[0] = h[0];
  for (let i = 1; i < n; i++) leftMax[i] = Math.max(leftMax[i - 1], h[i]);

  rightMax[n - 1] = h[n - 1];
  for (let i = n - 2; i >= 0; i--) rightMax[i] = Math.max(rightMax[i + 1], h[i]);

  let total = 0;
  for (let i = 0; i < n; i++) total += Math.min(leftMax[i], rightMax[i]) - h[i];
  return total;
}
// O(n) time, O(n) space.
```

**Step 2 — compress it.** You never need both maxima in full, only the smaller one — and
two pointers can track that:

```js
function trap(h) {
  let l = 0, r = h.length - 1, leftMax = 0, rightMax = 0, total = 0;
  while (l < r) {
    if (h[l] < h[r]) {
      leftMax = Math.max(leftMax, h[l]);
      total += leftMax - h[l];         // leftMax is the binding constraint here
      l++;
    } else {
      rightMax = Math.max(rightMax, h[r]);
      total += rightMax - h[r];
      r--;
    }
  }
  return total;
}
// O(n) time, O(1) space.
```

**Why it's correct:** when `h[l] < h[r]`, there is some bar at or before `r` that is at
least `h[r]`, which is bigger than `h[l]`. So the right-hand maximum for index `l` is
guaranteed to be ≥ `h[r]` — which means `min(leftMax, rightMax)` is definitely `leftMax`.
You can commit to the left side's water without ever computing the right max exactly.

**Showing the O(n)-space version first and then compressing it is a strong interview
move.** It demonstrates a derivation instead of a memorized trick.

### 5.9 The rest of the family

**Squares of a Sorted Array (LC 977)** — sorted but with negatives, so the biggest square
is at one of the two **ends**. Fill the output back to front:

```js
function sortedSquares(nums) {
  const n = nums.length, res = new Array(n);
  let l = 0, r = n - 1;
  for (let w = n - 1; w >= 0; w--) {
    const a = nums[l] * nums[l], b = nums[r] * nums[r];
    if (a > b) { res[w] = a; l++; }
    else       { res[w] = b; r--; }
  }
  return res;
}
// O(n) time, O(n) for the output (O(1) auxiliary).
// Sorting the squares would be O(n log n) — avoiding that is the point of the problem.
```

**Merge Sorted Array (LC 88) — fill from the back.** `nums1` holds `m` real values then
`n` empty slots. Merging left-to-right would overwrite values you haven't read yet;
merging **right-to-left** writes into slots that are already free:

```js
function merge(nums1, m, nums2, n) {
  let i = m - 1, j = n - 1, w = m + n - 1;
  while (j >= 0) {
    nums1[w--] = (i >= 0 && nums1[i] > nums2[j]) ? nums1[i--] : nums2[j--];
  }
}
// O(m + n) time, O(1) space.
```

Loop on `j >= 0` only: once `nums2` is used up, whatever remains in `nums1` is already in
place, so copying it is wasted work. **"Write backward to avoid clobbering"** is the
transferable lesson — it comes back in string problems and in-place shifting.

**Sort Colors / Dutch National Flag (LC 75)** — three pointers, one pass, sorting only
0s, 1s and 2s:

```js
function sortColors(nums) {
  let low = 0, mid = 0, high = nums.length - 1;
  while (mid <= high) {
    if (nums[mid] === 0) {
      [nums[low], nums[mid]] = [nums[mid], nums[low]];
      low++; mid++;
    } else if (nums[mid] === 1) {
      mid++;
    } else {                                            // it's a 2
      [nums[mid], nums[high]] = [nums[high], nums[mid]];
      high--;                                           // ⚠️ do NOT mid++ here
    }
  }
}
// O(n) time, O(1) space, one pass.
```

The regions the code maintains:

```
[ 0 0 0 | 1 1 1 | ? ? ? ? | 2 2 2 ]
         ↑       ↑         ↑
        low     mid       high
  done   done    UNSEEN     done
```

**The one trap, and it's the only thing interviewers ask about this problem:** when you
swap a 2 out to `high`, the value that comes back has **never been examined**, so you
must not advance `mid`. When you swap with `low`, the value coming back is guaranteed to
be a 1 (that region holds only 1s), which is why `mid++` is safe there. Explaining that
asymmetry proves you understand the algorithm.

Also note `mid <= high`, not `mid < high` — with `<` you leave the final element
unexamined. Test `[2,0,1]` and `[2,2]`.

### 5.10 Mistakes

| Mistake | Result |
|---|---|
| Moving the **taller** wall in Container With Most Water | Wrong answers on some inputs, and you can't explain why |
| `l <= r` when you need two distinct positions | Returns a "pair" made of one element used twice |
| Advancing `mid` after swapping from `high` in Sort Colors | Leaves unexamined values in the sorted region |
| `mid < high` instead of `mid <= high` | The last element is never examined |
| Forgetting `l < r` in the inner skip loops of Valid Palindrome | Pointer runs off the end |
| Merging forward in LC 88 | Overwrites values you still need |
| Sorting when the answer must be **original indexes** | You destroyed the indexes — instant fail |

---

## Part 6 — TRICK 2b: Sort First, Then Two Pointers
### Use it for triplets and quadruplets summing to a target (the k-sum family)

### 6.1 What it is

Two pointers needs sorted input. So **sort it yourself** (O(n log n)), then lock the first
element (or first two) with outer loops and run a normal two-pointer scan on the rest.

| How many numbers | Structure | Cost |
|---|---|---|
| 2 | one two-pointer scan | O(n) after sorting |
| 3 | 1 outer loop × scan | O(n²) |
| 4 | 2 outer loops × scan | O(n³) |
| k | (k−2) outer loops × scan | O(n^(k−1)) |

Sorting costs O(n log n), which is **smaller** than O(n²), so for 3Sum the sort disappears
from the complexity entirely.

### 6.2 The code — 3Sum (LC 15)

Find all **unique** triplets summing to zero.

```js
function threeSum(nums) {
  nums.sort((a, b) => a - b);            // O(n log n). ALWAYS pass the comparator.
  const res = [];
  const n = nums.length;

  for (let i = 0; i < n - 2; i++) {
    if (nums[i] > 0) break;                            // sorted: 3 positives can't sum to 0
    if (i > 0 && nums[i] === nums[i - 1]) continue;    // skip duplicate FIRST elements

    let l = i + 1, r = n - 1;
    while (l < r) {
      const sum = nums[i] + nums[l] + nums[r];
      if (sum < 0) l++;
      else if (sum > 0) r--;
      else {
        res.push([nums[i], nums[l], nums[r]]);
        while (l < r && nums[l] === nums[l + 1]) l++;   // skip duplicate SECOND elements
        while (l < r && nums[r] === nums[r - 1]) r--;   // skip duplicate THIRD elements
        l++; r--;
      }
    }
  }
  return res;
}
// Time  O(n²) — n outer steps × an O(n) scan. The sort is smaller, so it vanishes.
// Space O(1) extra (not counting the output, and the sort's own internal space).
```

### 6.3 The part that actually decides the interview: duplicates

The two-pointer scan is the easy bit. **The duplicate handling is the whole problem**, and
it's where interviews are lost. There are three separate skips, for three different
reasons:

| Skip | The guard | Why |
|---|---|---|
| First element | `if (i > 0 && nums[i] === nums[i-1]) continue;` | The same first value would regenerate every triplet you already found |
| Second element | `while (l < r && nums[l] === nums[l+1]) l++;` | Only **after** recording a hit — otherwise you'd skip valid distinct triplets |
| Third element | `while (l < r && nums[r] === nums[r-1]) r--;` | Same reason, from the right |

Two rules to keep straight:

1. The `i` skip compares **backward** to `i-1`, never forward to `i+1`. Comparing forward
   skips the *first* copy, which is the one you actually want to process.
2. The `l`/`r` skips live **inside** the `sum === 0` branch. Hoisting them out is a common
   wrong "optimization" that silently drops valid answers.

The `nums[i] > 0` break is a genuine speed win and free to justify: with the array sorted,
if the smallest of the three is positive the sum can't reach 0.

**If you blank on the skip logic under pressure:** collect triplets and key them into a
`Set` as `"a,b,c"` strings. Still O(n²) time, but up to O(n²) space, and it reads as an
admission you can't manage the pointers. Use it as a fallback only — and say that's what
you're doing.

### 6.4 Watch it run — `[-1, 0, 1, 2, -1, -4]`

Sorted: `[-4, -1, -1, 0, 1, 2]`

| `i` | `nums[i]` | The scan | Result |
|---|---|---|---|
| 0 | −4 | l=1(−1), r=5(2): sum −3 → too small, l++. l=2(−1): −3, l++. l=3(0): −2, l++. l=4(1): −1, l++ → pointers meet | nothing |
| 1 | −1 | l=2(−1), r=5(2): sum **0** ✅ record `[-1,-1,2]`. Then l=3(0), r=4(1): sum **0** ✅ record `[-1,0,1]` | 2 triplets |
| 2 | −1 | **skipped** — same as `nums[1]`, would repeat everything | — |
| 3 | 0 | l=4(1), r=5(2): sum 3 → r--, pointers meet | nothing |

Answer: `[[-1,-1,2], [-1,0,1]]`. Test `[0,0,0,0]` → exactly one triplet.

### 6.5 3Sum Closest (LC 16)

Return the sum closest to the target.

```js
function threeSumClosest(nums, target) {
  nums.sort((a, b) => a - b);
  const n = nums.length;
  let best = nums[0] + nums[1] + nums[2];       // ⭐ seed from REAL elements
  for (let i = 0; i < n - 2; i++) {
    let l = i + 1, r = n - 1;
    while (l < r) {
      const sum = nums[i] + nums[l] + nums[r];
      if (Math.abs(sum - target) < Math.abs(best - target)) best = sum;
      if (sum === target) return sum;           // exact — can't do better, exit early
      if (sum < target) l++;
      else r--;
    }
  }
  return best;
}
// O(n²) time, O(1) space.
```

**No duplicate handling needed** — you're returning a single number, not a set of distinct
triplets. But **initialize `best` from actual elements**, never from `0` or `Infinity`:
`Infinity` breaks the `Math.abs(best - target)` arithmetic, and `0` is simply a wrong
answer when no sum is near zero.

### 6.6 3Sum Smaller (LC 259) — the counting trick worth stealing

Count triplets whose sum is `< target`.

```js
function threeSumSmaller(nums, target) {
  nums.sort((a, b) => a - b);
  let count = 0;
  for (let i = 0; i < nums.length - 2; i++) {
    let l = i + 1, r = nums.length - 1;
    while (l < r) {
      if (nums[i] + nums[l] + nums[r] < target) {
        count += r - l;      // ⭐ every index between l and r also works — the array is sorted
        l++;
      } else {
        r--;
      }
    }
  }
  return count;
}
// O(n²) time, O(1) space.
```

`count += r - l` is the reusable idea: **when a sorted pair satisfies a bound, every pair
between them satisfies it too**, so you count a whole block in one step instead of
looping over it. The identical trick appears in Subarray Product Less Than K (Part 7.11).
Recognize it as one idea, not two.

### 6.7 4Sum (LC 18) and the general k-sum

```js
function fourSum(nums, target) {
  nums.sort((a, b) => a - b);
  const n = nums.length, res = [];
  for (let i = 0; i < n - 3; i++) {
    if (i > 0 && nums[i] === nums[i - 1]) continue;
    for (let j = i + 1; j < n - 2; j++) {
      if (j > i + 1 && nums[j] === nums[j - 1]) continue;    // ⚠️ j > i+1, not j > 0
      let l = j + 1, r = n - 1;
      while (l < r) {
        const sum = nums[i] + nums[j] + nums[l] + nums[r];
        if (sum < target) l++;
        else if (sum > target) r--;
        else {
          res.push([nums[i], nums[j], nums[l], nums[r]]);
          while (l < r && nums[l] === nums[l + 1]) l++;
          while (l < r && nums[r] === nums[r - 1]) r--;
          l++; r--;
        }
      }
    }
  }
  return res;
}
// O(n³) time, O(1) auxiliary space.
```

The second guard is `j > i + 1`, **not** `j > 0` — dedup compares against the previous
value *within this level's range*, and level `j` starts at `i+1`.

**The sentence to have ready when they ask "can you generalize?":**
> *"k-sum on a sorted array is O(n^(k−1)): fix k−2 elements with nested loops, then run a
> linear two-pointer scan on the rest."*

For very large `k` you'd switch to hashing pair sums (O(n²) space). Mention it exists;
don't implement it unless asked.

### 6.8 Mistakes

| Mistake | Result |
|---|---|
| `nums.sort()` without `(a,b) => a-b` | Sorts as **text**: `[10, 9, 1]` → `[1, 10, 9]` |
| `(a, b) => a > b` as a comparator | A boolean isn't a comparator (never returns −1) → subtly wrong order |
| Comparing `nums[i] === nums[i+1]` for the outer skip | Skips the first copy instead of the repeats |
| Hoisting the `l`/`r` skips out of the hit branch | Drops valid triplets |
| Seeding `best = 0` or `Infinity` in 3Sum Closest | Wrong answer / broken arithmetic |
| Not mentioning that `sort()` mutates the caller's array | Fine on LeetCode; a real bug in code review |

---
## Part 7 — TRICK 3: Sliding Window
### Use it when the answer is a continuous chunk and you want the longest / shortest / max / min

**This is the highest-value pattern in the phase, and the most common one in frontend
interviews.** Days 6 and 7 exist for it.

### 7.1 What a "window" actually is

A **window** is a continuous chunk of the array, marked by two indexes: `l` (left edge)
and `r` (right edge). Everything from `l` to `r` inclusive is "inside the window".

```
arr =  [ 4 , 2 , 9 , 7 , 1 , 5 ]
index    0   1   2   3   4   5
             ╔═══════════╗
             ║ 2   9   7 ║        ← the window: l = 1, r = 3
             ╚═══════════╝
             window length = r − l + 1 = 3 − 1 + 1 = 3
```

**Sliding** it means moving it along without rebuilding it. To move one step right you do
exactly two things:

```
before:   [ 4 ,║ 2 , 9 , 7 ║, 1 , 5 ]     sum = 18
after:    [ 4 , 2 ,║ 9 , 7 , 1 ║, 5 ]     sum = 18 + 1 − 2 = 17
                                                     ↑      ↑
                                             1 entered    2 left
```

**One addition, one subtraction.** You never re-add the middle. That is the entire idea,
and it's why a window is O(n) instead of O(n²).

The physical picture: you're looking through a train window at a row of houses. As the
train moves, one new house comes into view and one old house leaves. You don't re-count
the houses in the middle.

### 7.2 The two conditions — check these BEFORE you commit

A window only works if **both** are true. Skip this check and you'll write code that
silently returns wrong answers instead of crashing, which is far worse.

**Condition 1 — the answer must be contiguous.**
Subarray ✅ Substring ✅ Subsequence ❌ (skipping allowed → that's Dynamic Programming,
Phase 10).

**Condition 2 — validity must be "monotone".**
Growing the window on the right can only push it *toward* invalid; shrinking from the left
can only push it *toward* valid. Nothing may flip back and forth.

> **The test to run in your head:** *"If my window is currently invalid, could adding
> another element on the right ever make it valid again?"*
> If yes → **it is not a window problem.**

The classic failure: *"shortest subarray with sum ≥ target"* **when negatives are allowed**
(LC 862). Adding an element can *decrease* the sum, so growing doesn't reliably help and
shrinking doesn't reliably hurt. That one needs prefix sums plus a monotonic deque.

**Knowing when the pattern doesn't apply is a senior-level signal.** Junior candidates
throw a window at anything containing the word "subarray".

### 7.3 Which of the three templates do I need?

Ask two questions, in this order:

```
Does the problem give me the window size ("of size k")?
├── YES → Template A (fixed window)
└── NO  → Am I looking for the LONGEST or the SHORTEST valid chunk?
          ├── LONGEST  → Template B
          └── SHORTEST → Template C
```

Decide this **before** writing anything. Deciding afterwards is how people put the
recording line in the wrong place, which produces answers that pass the sample test and
fail the hidden ones.

### 7.4 Why a window is O(n) even though there's a loop inside a loop

**Say this unprompted in every window interview.** "There's a nested loop, so it's O(n²)"
is the single most common complexity mistake in this phase, and interviewers listen for it
specifically.

> *"`r` advances exactly n times. `l` only ever moves forward, and across the whole run it
> can move at most n times **in total** — not n times per step. So total pointer movement
> is at most 2n, which is O(n)."*

The general rule (from Phase 0):

| Inner loop counter | Effect | Result |
|---|---|---|
| **Resets** each outer iteration | multiply | O(n²) |
| **Persists**, only moves forward | add | O(n) |

In a sliding window, `l` persists — it never goes back to 0. So the inner `while` **adds**
up to n steps across the entire run instead of multiplying.

### 7.5 Template A — fixed window (size k)

**Use when:** "subarray of size k", "every window of k", "average of every k".

```js
function fixedWindow(arr, k) {
  if (k > arr.length) return 0;

  let sum = 0;
  for (let i = 0; i < k; i++) sum += arr[i];      // build the first window
  let best = sum;

  for (let i = k; i < arr.length; i++) {
    sum += arr[i] - arr[i - k];                   // ⭐ THE SLIDE: add newcomer, drop leaver
    best = Math.max(best, sum);
  }
  return best;
}
// O(n) time, O(1) space.
```

`sum += arr[i] - arr[i-k]` **is** the slide. If you can write that line without
hesitating, fixed windows are done.

**Watch it run** — `[2, 1, 5, 1, 3, 2]`, k = 3:

| Step | Window | Calculation | `sum` | `best` |
|---|---|---|---|---|
| build | `[2,1,5]` | 2+1+5 | 8 | 8 |
| i=3 | `[1,5,1]` | 8 + arr[3] − arr[0] = 8 + 1 − 2 | 7 | 8 |
| i=4 | `[5,1,3]` | 7 + arr[4] − arr[1] = 7 + 3 − 1 | **9** | 9 |
| i=5 | `[1,3,2]` | 9 + arr[5] − arr[2] = 9 + 2 − 5 | 6 | 9 |

Answer: **9**.

**The compact one-loop version**, and the three bugs it hides:

```js
for (let i = 0; i < arr.length; i++) {
  sum += arr[i];                                  // add the entering element
  if (i >= k)     sum -= arr[i - k];              // remove the leaving element
  if (i >= k - 1) best = Math.max(best, sum);     // record only COMPLETE windows
}
```

1. The element that leaves is `arr[i - k]` — **not** `arr[i-k+1]`.
2. `if (i >= k - 1)` gates the recording. Recording earlier records a *partial* window,
   which with positive numbers is often larger than any real window — a wrong answer that
   looks completely plausible.
3. Guard `k > arr.length`, or you return `-Infinity`.

Under interview pressure, **prefer the two-phase version**. It's harder to get wrong.

**Maximum Average Subarray I (LC 643)** — the same thing, with one refinement:

```js
function findMaxAverage(nums, k) {
  let sum = 0;
  for (let i = 0; i < k; i++) sum += nums[i];
  let best = sum;
  for (let i = k; i < nums.length; i++) {
    sum += nums[i] - nums[i - k];
    best = Math.max(best, sum);
  }
  return best / k;                 // ⭐ divide ONCE, at the end
}
// O(n) time, O(1) space.
```

Dividing once at the end instead of per window avoids accumulating floating-point error
and is one fewer operation per iteration. Small, but it reads as care.

### 7.6 Template B — variable window, LONGEST

**Use when:** "longest substring/subarray such that…".

```js
function longestValid(arr) {
  let l = 0, best = 0;
  for (let r = 0; r < arr.length; r++) {
    // 1. add arr[r] to the window
    while (/* the window is INVALID */) {
      // 2. remove arr[l] from the window
      l++;
    }
    // 3. the window is now guaranteed valid → record it
    best = Math.max(best, r - l + 1);
  }
  return best;
}
// O(n) time. Space = whatever you use to track what's inside the window.
```

**Watch it run** — Longest Substring Without Repeating Characters (LC 3), `"abcabcbb"`:

```js
function lengthOfLongestSubstring(s) {
  const seen = new Set();
  let l = 0, best = 0;
  for (let r = 0; r < s.length; r++) {
    while (seen.has(s[r])) {      // invalid: this character is already inside
      seen.delete(s[l]);
      l++;
    }
    seen.add(s[r]);
    best = Math.max(best, r - l + 1);
  }
  return best;
}
// O(n) time — each character is added once and deleted at most once.
// O(min(n, alphabet size)) space.
```

| `r` | char | Duplicate? | Shrink | Window | Length | `best` |
|---|---|---|---|---|---|---|
| 0 | a | no | – | `a` | 1 | 1 |
| 1 | b | no | – | `ab` | 2 | 2 |
| 2 | c | no | – | `abc` | **3** | 3 |
| 3 | a | yes | drop `a`, l→1 | `bca` | 3 | 3 |
| 4 | b | yes | drop `b`, l→2 | `cab` | 3 | 3 |
| 5 | c | yes | drop `c`, l→3 | `abc` | 3 | 3 |
| 6 | b | yes | drop `a` (l→4), drop `b` (l→5) | `cb` | 2 | 3 |
| 7 | b | yes | drop `c` (l→6), drop `b` (l→7) | `b` | 1 | 3 |

Answer: **3**.

Look at step 6: **one entering character forced two removals.** That is exactly why the
inner loop must be `while` and not `if`.

⚠️ **Order matters:** shrink **before** adding `s[r]`. If you add first, you'll delete the
character you just inserted and loop forever on `"aa"`. Dry-run `"aa"` and `"abba"` before
claiming it works.

**The "jump instead of step" variant**, which interviewers ask for when they say "can you
avoid the inner loop?":

```js
function lengthOfLongestSubstringMap(s) {
  const last = new Map();
  let l = 0, best = 0;
  for (let r = 0; r < s.length; r++) {
    const c = s[r];
    if (last.has(c)) l = Math.max(l, last.get(c) + 1);   // ⚠️ Math.max is NOT optional
    last.set(c, r);
    best = Math.max(best, r - l + 1);
  }
  return best;
}
// O(n) time, O(min(n, alphabet)) space.
```

**The `Math.max` is the whole trap.** The map also holds indexes of characters that have
already left the window, so a stale index can be *behind* `l`. Without the clamp, `l`
moves **backward** and the window grows past a duplicate.

Test `"abba"`: at the final `a`, `last.get('a')` is 0, so without `Math.max` `l` becomes 1
when it should stay at 2 — and you return 3 instead of 2. The `Set` version has no such
hazard, which is why it's the better one to write live.

### 7.7 Template C — variable window, SHORTEST

**Use when:** "shortest/minimum subarray such that…".

```js
function shortestValid(arr) {
  let l = 0, best = Infinity;
  for (let r = 0; r < arr.length; r++) {
    // 1. add arr[r] to the window
    while (/* the window is VALID */) {
      best = Math.min(best, r - l + 1);   // ⭐ record BEFORE you shrink validity away
      // 2. remove arr[l] from the window
      l++;
    }
  }
  return best === Infinity ? 0 : best;
}
```

**Watch it run** — Minimum Size Subarray Sum (LC 209), target = 7, `[2,3,1,2,4,3]`:

```js
function minSubArrayLen(target, nums) {
  let l = 0, sum = 0, best = Infinity;
  for (let r = 0; r < nums.length; r++) {
    sum += nums[r];
    while (sum >= target) {                // valid → try shrinking to get shorter
      best = Math.min(best, r - l + 1);
      sum -= nums[l];
      l++;
    }
  }
  return best === Infinity ? 0 : best;
}
// O(n) time, O(1) space.
```

| `r` | added | `sum` | ≥ 7? | Action | `l` | `best` |
|---|---|---|---|---|---|---|
| 0 | 2 | 2 | no | – | 0 | ∞ |
| 1 | 3 | 5 | no | – | 0 | ∞ |
| 2 | 1 | 6 | no | – | 0 | ∞ |
| 3 | 2 | 8 | yes | record length 4, drop 2 → sum 6 | 1 | 4 |
| 4 | 4 | 10 | yes | record length 4, drop 3 → sum 7 | 2 | 4 |
|  |  | 7 | yes | record length **3**, drop 1 → sum 6 | 3 | 3 |
| 5 | 3 | 9 | yes | record length 3, drop 2 → sum 7 | 4 | 3 |
|  |  | 7 | yes | record length **2**, drop 4 → sum 3 | 5 | 2 |

Answer: **2** (the subarray `[4,3]`).

**Say the precondition out loud:** *"this needs all values to be positive."* That's
condition 2 from Part 7.2, and volunteering it shows you know where the pattern breaks.

### 7.8 The one table that separates B from C

**Burn this in.** Same skeleton, mirrored:

|  | LONGEST (Template B) | SHORTEST (Template C) |
|---|---|---|
| Shrink while the window is… | **INVALID** | **VALID** |
| Record the answer… | **after** the inner loop | **inside** the inner loop |
| Start `best` at | `0` | `Infinity` |
| If no answer exists | already `0` | convert `Infinity → 0` at the end |

That's it. Almost every window problem is one of these two with a different notion of
"what's in the window" and "valid". **Getting the recording position backwards is the #1
window bug** — it passes the sample test and fails the hidden ones.

**Why `while` and not `if`:** one entering element can force several removals (a big
number entering a sum-bounded window may require dropping several small ones). An `if`
shrinks at most once per step and quietly produces wrong answers.

### 7.9 What to keep inside the window (choosing the "state")

When the window is more complicated than a running sum, you need something to remember
what's inside it:

| Keep this | Use it when | Cost |
|---|---|---|
| a plain number (sum / product) | the rule is numeric ("sum ≥ target") | O(1) |
| a `Set` | you only need "is it in here?" ("no duplicates") | O(k) space |
| a `Map` | you need **counts**, or last-seen positions | O(k) space |
| `new Array(26).fill(0)` | lowercase letters only | O(1) space, fastest |
| `new Array(128).fill(0)` or `Uint8Array(128)` | any ASCII character | O(1) space |

⚠️ **Never write `Object.keys(obj).length` inside the loop** to count distinct items —
that's O(k) *every single step* and turns your O(n) into O(n·k). Use `map.size`, which is
O(1).

**At most k distinct characters (LC 340; Fruit Into Baskets LC 904 is k = 2):**

```js
function atMostKDistinct(s, k) {
  const count = new Map();
  let l = 0, best = 0;
  for (let r = 0; r < s.length; r++) {
    count.set(s[r], (count.get(s[r]) || 0) + 1);
    while (count.size > k) {                          // map.size is O(1) ⭐
      const out = s[l];
      const c = count.get(out) - 1;
      if (c === 0) count.delete(out); else count.set(out, c);   // ⭐ delete at zero
      l++;
    }
    best = Math.max(best, r - l + 1);
  }
  return best;
}
// O(n) time, O(k) space.
```

**The most common bug in this whole family:** forgetting to `delete` the key when its
count reaches zero. Leave it in, and `count.size` keeps counting characters that already
left the window, so the constraint never triggers.

**At most k zeros (LC 1004, Max Consecutive Ones III):**

```js
function longestOnes(nums, k) {
  let l = 0, zeros = 0, best = 0;
  for (let r = 0; r < nums.length; r++) {
    if (nums[r] === 0) zeros++;
    while (zeros > k) {
      if (nums[l] === 0) zeros--;
      l++;
    }
    best = Math.max(best, r - l + 1);
  }
  return best;
}
// O(n) time, O(1) space.
```

**Identical skeleton, only the state changed.** That's the point. LC 3, LC 424, LC 904 and
LC 1004 are one problem with four different notions of "invalid" — if that isn't obvious
yet, write all four back to back.

### 7.10 Longest Repeating Character Replacement (LC 424) — and the honest version

You may replace up to `k` characters. The window is valid when
`windowLength − (count of the most common letter in it) <= k` — keep the most common
letter, replace everything else.

```js
function characterReplacement(s, k) {
  const count = new Array(26).fill(0);
  const idx = c => c.charCodeAt(0) - 65;             // ⚠️ 65 = 'A' — LC 424 is UPPERCASE
  let l = 0, best = 0;
  for (let r = 0; r < s.length; r++) {
    count[idx(s[r])]++;
    while (r - l + 1 - Math.max(...count) > k) {     // recompute inside the condition
      count[idx(s[l])]--;
      l++;
    }
    best = Math.max(best, r - l + 1);
  }
  return best;
}
// O(26n) = O(n) time, O(1) space (26 counters).
// Math.max over a FIXED 26-element array is O(1) — safe precisely because 26 is a constant.
```

⚠️ **The offset trap, and it's nasty.** `charCodeAt(0) - 97` maps `'a'` → 0;
`- 65` maps `'A'` → 0. LC 438 and 567 are lowercase; **LC 424 is uppercase**. Use the wrong
offset and you index `count[-32]`, which is `undefined` → `Math.max(...count)` is `NaN` →
every comparison with `NaN` is false → the window never shrinks → you get a silently wrong
answer instead of a crash. **Read the constraints for case before picking the offset.** If
the input can be mixed case, index by the raw code into `new Array(128).fill(0)`.

**The subtlety you'll see in every online solution.** The widely-copied version never
decrements a cached `maxFreq`, and uses `if` instead of `while`:

```js
let maxFreq = 0;
for (let r = 0; r < s.length; r++) {
  count[idx(s[r])]++;
  maxFreq = Math.max(maxFreq, count[idx(s[r])]);   // never decreases
  if (r - l + 1 - maxFreq > k) { count[idx(s[l])]--; l++; }
  best = Math.max(best, r - l + 1);
}
```

It returns the right answer, and the reason is not obvious: `maxFreq` may be stale (too
large), which makes the validity check too *permissive*, so the window can genuinely be
invalid at some steps. But the window size never shrinks — each step either grows it by
one or holds it — and it can only grow when the check passes. A window larger than the
true answer would need a genuinely larger `maxFreq` than any real window contains, which
never happens. So the recorded maximum is never bigger than some genuinely valid window.

**What to do in an interview: write the recompute version.** It's the same O(n), it needs
no argument, and if the interviewer raises the cached trick you can explain why that also
works. Reciting a memorized trick whose correctness you can't defend is worse than a clean
solution.

### 7.11 Counting subarrays — the `r − l + 1` trick

When the question is **"how many subarrays satisfy X"** rather than "how long is the
longest", the number of valid subarrays **ending at `r`** is exactly the current window
length.

Why: if the window is `l..r`, the valid chunks ending at `r` are
`[l..r], [l+1..r], … [r..r]` — that's `r − l + 1` of them.

```js
// Subarray Product Less Than K (LC 713) — needs all values ≥ 1
function numSubarrayProductLessThanK(nums, k) {
  if (k <= 1) return 0;                 // no product of positive ints is below 1
  let l = 0, prod = 1, count = 0;
  for (let r = 0; r < nums.length; r++) {
    prod *= nums[r];
    while (prod >= k) { prod /= nums[l]; l++; }
    count += r - l + 1;                 // ⭐ every chunk ending at r that starts in [l, r]
  }
  return count;
}
// O(n) time, O(1) space.
```

The `k <= 1` guard also stops `l` running past `r`. This is the same insight as
`count += r - l` in 3Sum Smaller (Part 6.6) — one idea, two problems.

Repeated float division is a legitimate concern here (accumulated error). It's fine for
LeetCode limits; if an interviewer pushes, offer log-sums or a prefix-product-plus-binary-
search alternative and note the precision trade.

### 7.12 "Exactly k" — the subtract trick

There is no clean single window for "exactly k distinct", because that rule isn't monotone
(Part 7.2). The fix is subtraction:

> **exactly k = (at most k) − (at most k−1)**

```js
// Subarrays with K Different Integers (LC 992)
function subarraysWithKDistinct(nums, k) {
  const atMost = (m) => {
    const count = new Map();
    let l = 0, total = 0;
    for (let r = 0; r < nums.length; r++) {
      count.set(nums[r], (count.get(nums[r]) || 0) + 1);
      while (count.size > m) {
        const c = count.get(nums[l]) - 1;
        if (c === 0) count.delete(nums[l]); else count.set(nums[l], c);
        l++;
      }
      total += r - l + 1;
    }
    return total;
  };
  return atMost(k) - atMost(k - 1);
}
// O(n) time (two passes), O(k) space.
```

Whenever you see the word **"exactly"**, reach for this. It also converts "sum equals k"
into "at most k minus at most k−1" for positive arrays, and it's the standard trick for
"exactly k odd numbers" (LC 1248). **Two easy passes beat one clever pass you can't debug
live.**

### 7.13 Two harder window problems worth working through

**Find All Anagrams in a String (LC 438)** — fixed window + letter counts.

The version to write live:

```js
function findAnagramsSimple(s, p) {
  const res = [], k = p.length;
  if (k > s.length) return res;
  const need = new Array(26).fill(0), win = new Array(26).fill(0);
  const idx = c => c.charCodeAt(0) - 97;              // 'a' → 0
  for (const c of p) need[idx(c)]++;

  for (let r = 0; r < s.length; r++) {
    win[idx(s[r])]++;
    if (r >= k) win[idx(s[r - k])]--;                 // the leaver
    if (r >= k - 1 && need.every((v, i) => v === win[i])) res.push(r - k + 1);
  }
  return res;
}
// O(26n) = O(n) time, O(1) space.
```

Say the complexity as *"O(26n), which is O(n) — the 26 is a constant because the alphabet
is fixed."* That sentence proves you know the difference between a constant and a variable.

The O(1)-per-step version, if they push for it — it tracks how many letters currently
agree instead of comparing both arrays:

```js
function findAnagrams(s, p) {
  const res = [];
  if (p.length > s.length) return res;
  const need = new Array(26).fill(0), win = new Array(26).fill(0);
  const idx = c => c.charCodeAt(0) - 97;
  for (const c of p) need[idx(c)]++;

  let matches = 0;
  for (let i = 0; i < 26; i++) if (need[i] === 0) matches++;

  for (let r = 0; r < s.length; r++) {
    const inC = idx(s[r]);
    win[inC]++;
    if (win[inC] === need[inC]) matches++;
    else if (win[inC] === need[inC] + 1) matches--;   // just broke a previous match

    if (r >= p.length) {
      const outC = idx(s[r - p.length]);
      win[outC]--;
      if (win[outC] === need[outC]) matches++;
      else if (win[outC] === need[outC] - 1) matches--;
    }

    if (r >= p.length - 1 && matches === 26) res.push(r - p.length + 1);
  }
  return res;
}
// O(n) time, O(1) space.
```

**Permutation in String (LC 567)** is this exact problem returning `true` on the first hit.

**Minimum Window Substring (LC 76)** — the hardest problem in the phase. Template C with
counts:

```js
function minWindow(s, t) {
  if (t.length > s.length) return '';
  const need = new Map();
  for (const c of t) need.set(c, (need.get(c) || 0) + 1);

  let required = need.size;      // how many distinct chars still need satisfying
  let formed = 0;                // how many are currently satisfied
  const win = new Map();
  let l = 0, bestLen = Infinity, bestStart = 0;

  for (let r = 0; r < s.length; r++) {
    const c = s[r];
    if (need.has(c)) {
      win.set(c, (win.get(c) || 0) + 1);
      if (win.get(c) === need.get(c)) formed++;     // === not >=, so it counts exactly once
    }

    while (formed === required) {                    // valid → shrink
      if (r - l + 1 < bestLen) { bestLen = r - l + 1; bestStart = l; }
      const out = s[l];
      if (need.has(out)) {
        win.set(out, win.get(out) - 1);
        if (win.get(out) < need.get(out)) formed--;
      }
      l++;
    }
  }
  return bestLen === Infinity ? '' : s.slice(bestStart, bestStart + bestLen);
}
// O(|s| + |t|) time and space.
```

Three details that decide whether it works:

1. Track a **counter** (`formed`), don't compare whole maps every step.
2. `if (win.get(c) === need.get(c)) formed++` uses `===`, **not** `>=`. With `>=`, extra
   copies of a character inflate `formed` past `required` and you emit garbage windows.
3. Record `bestStart`/`bestLen` and call `slice` **once at the very end**. Slicing inside
   the loop makes it O(n²) — the trap from Part 3.5.

Test `s="a", t="aa"` → `""` (duplicates in `t` matter) and `s="ab", t="b"` → `"b"`.

**If you can write this from the Template C skeleton without looking, variable windows are
genuinely done.**

### 7.14 When sliding window is the WRONG tool

| Problem | Why the window fails | What to use instead |
|---|---|---|
| Shortest subarray with sum ≥ k, **negatives allowed** (LC 862) | Sum isn't monotone in window size | Prefix sums + monotonic deque |
| Subarray sum equals k, **negatives allowed** (LC 560) | Same reason | **Prefix sums + hash map** (Part 8) |
| Longest **subsequence** (skipping allowed) | Not contiguous | Dynamic Programming (Phase 10) |
| Maximum in every window of size k (LC 239) | You can't remove the max in O(1) when it leaves | Monotonic deque (Phase 6) |
| Count binary substrings (LC 696) | Validity flips at every run boundary | Run-length grouping (practice problem 31) |

**The general rule:** whatever you keep as window state must support **removal from the
left in O(1)**. Sums can. Counts can. A **maximum cannot** — removing the current max
means knowing the next-largest, which you didn't store. That's why "sliding window
maximum" needs a deque and lands in Phase 6.

### 7.15 Mistakes

| Mistake | How you'll notice |
|---|---|
| `if` instead of `while` for the shrink | Wrong answers when one newcomer forces multiple removals |
| Recording in the wrong place (B vs C) | Sample test passes, hidden tests fail |
| Adding `s[r]` before shrinking in LC 3 | Infinite loop on `"aa"` |
| `r - l` instead of `r - l + 1` | Every length is off by one |
| `arr.slice(l, r+1)` inside the loop to "get the window" | Turns O(n) into **O(n²)** |
| Not deleting map keys when the count hits 0 | `map.size` never shrinks, constraint never fires |
| Wrong `charCodeAt` offset (97 vs 65) | Silent `NaN`, window never shrinks |
| Saying "nested loop, so O(n²)" | Wrong, and interviewers listen for it — see Part 7.4 |

---
## Part 8 — TRICK 4: Prefix Sums
### Use it when you need sums over ranges — especially when negative numbers are allowed

### 8.1 What it is — one sentence

> **A range sum is the difference of two running totals.**

Build an array where `pre[i]` = "the sum of the first `i` elements":

```
nums  =      [ 1 ,  2 ,  3 ,  4 ]
pre   =  [ 0 ,  1 ,  3 ,  6 , 10 ]
           ↑    ↑    ↑    ↑    ↑
        sum of  1    2    3    4  items
        0 items
```

Now **any** range sum is one subtraction:

```
sum of nums[1..2]  (that's 2 + 3 = 5)
  =  pre[3] − pre[1]
  =    6    −   1     =  5   ✅
```

Instead of looping over the range for every question (O(n) each), you answer in O(1).

Everything in this Part is that one sentence applied to different value types (sums,
products, remainders, balances) and different dimensions.

### 8.2 The build, and the `+1` convention that kills off-by-one bugs

```js
function buildPrefix(nums) {
  const pre = new Array(nums.length + 1).fill(0);   // ⭐ length n+1, and pre[0] = 0
  for (let i = 0; i < nums.length; i++) pre[i + 1] = pre[i] + nums[i];
  return pre;
}
// O(n) time, O(n) space.

// Sum of nums[i..j], both ends INCLUDED:
const rangeSum = (pre, i, j) => pre[j + 1] - pre[i];
```

**Always use the `n+1` form with `pre[0] = 0`.** The alternative (`pre[i]` = sum up to and
including `i`) forces a special case for `i === 0` in every single query, and that special
case is exactly where the bug goes. With the `n+1` convention, `pre[k]` simply means "the
sum of the first `k` elements" — no element sits at a confusing index, and the empty
prefix has a real slot.

**Sanity check to run in your head:** `rangeSum(pre, 0, n-1)` must equal `pre[n] - pre[0]`
= the total of the whole array. If your formula doesn't produce that, your convention is
wrong.

### 8.3 Build the array, or just keep a running number?

| Situation | Do this |
|---|---|
| Many range questions, array never changes | Build the array once: O(n) build, O(1) per question |
| One pass, you only ever need "sum so far" | Keep **one variable** — O(1) space |
| The array changes between questions | Prefix sums are the wrong tool → Fenwick / segment tree (out of scope) |

**Most interview problems in this phase are the middle case.** You never build the array at
all — you keep one `sum` variable and a hash map. Building an array you scan only once is
O(n) space you didn't need, and an interviewer may well ask you to remove it.

**Range Sum Query — Immutable (LC 303)** is the first case, and the reason the class exists:

```js
class NumArray {
  constructor(nums) {
    this.pre = new Array(nums.length + 1).fill(0);
    for (let i = 0; i < nums.length; i++) this.pre[i + 1] = this.pre[i] + nums[i];
  }
  sumRange(i, j) { return this.pre[j + 1] - this.pre[i]; }   // O(1)
}
// Construction O(n) time and space; each query O(1).
```

The framing that matters: *"this trades O(n) preprocessing for O(1) queries — worth it
when queries outnumber builds, which is the whole point of the 'immutable' variant."*

### 8.4 The most important problem here: Subarray Sum Equals K (LC 560)

Count how many subarrays add up to exactly `k`. **Negative numbers are allowed**, which is
precisely why sliding window cannot do this.

```js
function subarraySum(nums, k) {
  const seen = new Map([[0, 1]]);      // ⭐ seed: one "empty prefix" with value 0
  let sum = 0, count = 0;
  for (const x of nums) {
    sum += x;
    count += seen.get(sum - k) || 0;                  // ⭐ check BEFORE inserting
    seen.set(sum, (seen.get(sum) || 0) + 1);
  }
  return count;
}
// O(n) time, O(n) space.
```

**The derivation — say this out loud:**

> A subarray ending at position `r` sums to `k` exactly when `pre[r] − pre[i] = k`, which
> rearranges to `pre[i] = pre[r] − k`. So while scanning, I ask: *how many earlier prefixes
> equal `sum − k`?* I need **counts**, not just yes/no, so it's a `Map` of counts, not a
> `Set`.

**Watch it run** — `[1, 2, 3]`, k = 3:

| element | `sum` | looking for `sum − k` | found? | `count` | map after |
|---|---|---|---|---|---|
| start | 0 | – | – | 0 | `{0:1}` |
| 1 | 1 | −2 | no | 0 | `{0:1, 1:1}` |
| 2 | 3 | 0 | **yes ×1** | 1 | `{0:1, 1:1, 3:1}` |
| 3 | 6 | 3 | **yes ×1** | **2** | `{0:1, 1:1, 3:1, 6:1}` |

Answer: **2** — the subarrays `[1,2]` and `[3]`.

**Two details that break it if you get them wrong:**

1. **Seed the map with `[0, 1]`.** Without it you miss every subarray that starts at index
   0 — in the trace above, `[1,2]` would be missed.
2. **Check before inserting.** With `k === 0`, inserting first lets the current prefix
   match itself and you'd count an empty subarray. Test `[1,-1,0]` with `k=0` → 3.

### 8.5 The variants — same map, different policy

**Longest subarray summing to k (LC 325)** — store the **first** index for each prefix and
never overwrite:

```js
function maxSubArrayLen(nums, k) {
  const firstIdx = new Map([[0, -1]]);              // prefix 0 happens "before" index 0
  let sum = 0, best = 0;
  for (let i = 0; i < nums.length; i++) {
    sum += nums[i];
    if (firstIdx.has(sum - k)) best = Math.max(best, i - firstIdx.get(sum - k));
    if (!firstIdx.has(sum)) firstIdx.set(sum, i);   // ⭐ keep the EARLIEST occurrence only
  }
  return best;
}
// O(n) time, O(n) space.
```

`if (!has)` before `set` is the whole trick. For a **longest** answer you want the earliest
matching prefix; overwriting would give you the shortest. **Same map, opposite policy** —
and yes, if the problem asked for the *shortest* such subarray you'd always overwrite.

**Subarray sums divisible by k (LC 974)** — two prefixes sharing a remainder means the
chunk between them is divisible by `k`:

```js
function subarraysDivByK(nums, k) {
  const count = new Array(k).fill(0);
  count[0] = 1;
  let sum = 0, total = 0;
  for (const x of nums) {
    sum += x;
    const r = ((sum % k) + k) % k;      // ⚠️ JS % is REMAINDER, not modulo
    total += count[r];
    count[r]++;
  }
  return total;
}
// O(n + k) time, O(k) space.
```

⚠️ **The JavaScript trap.** `%` in JS takes the sign of the **left** operand:

```js
-1 % 5;              // -1   ← in Python this is 4
-7 % 3;              // -1
((-1 % 5) + 5) % 5;  //  4   ← the normalization you must write
```

Without `((x % k) + k) % k`, negative remainders index outside your array (`count[-1]` is
`undefined`, so `total += undefined` → `NaN`), or land in a different bucket than the
equivalent positive remainder. It gives you `NaN` rather than throwing. This is a genuine
JS-versus-Python difference that catches people who learned the pattern in Python.
**Memorize `((x % k) + k) % k` as one unit.**

**Equal number of 0s and 1s (LC 525)** — reframe it: turn every `0` into `−1`, and "equal
counts" becomes "the running total returns to a value it had before":

```js
function findMaxLength(nums) {
  const firstIdx = new Map([[0, -1]]);
  let balance = 0, best = 0;
  for (let i = 0; i < nums.length; i++) {
    balance += nums[i] === 1 ? 1 : -1;
    if (firstIdx.has(balance)) best = Math.max(best, i - firstIdx.get(balance));
    else firstIdx.set(balance, i);
  }
  return best;
}
// O(n) time, O(n) space.
```

The reframing **"equal counts of two things" → "a signed running total revisits a value"**
generalizes to any two-way balance: matched parentheses depth, upvotes/downvotes,
gains/losses. Whenever a problem says "equal number of X and Y", reach for ±1 and a prefix
map.

**Product of Array Except Self (LC 238)** — prefix and suffix **products**:

```js
function productExceptSelf(nums) {
  const n = nums.length;
  const res = new Array(n).fill(1);

  let prefix = 1;
  for (let i = 0; i < n; i++) { res[i] = prefix; prefix *= nums[i]; }        // everything LEFT of i

  let suffix = 1;
  for (let i = n - 1; i >= 0; i--) { res[i] *= suffix; suffix *= nums[i]; }  // × everything RIGHT of i

  return res;
}
// O(n) time, O(1) auxiliary space — the output array doesn't count, and you should SAY that,
// because it is exactly the constraint the problem is testing.
```

Two things to volunteer:

- **Why not just divide the total by `nums[i]`?** It breaks on a single zero (division by
  zero) and on two zeros (every answer is 0, but so is the total). The problem forbids
  division for that reason. Handling zeros with division would mean counting them and
  special-casing 0, 1, and ≥2 zeros — mention it, then write the prefix/suffix version.
- **Why two passes into one array?** Writing prefixes on the way up and multiplying
  suffixes on the way down uses the output as your scratch space — that's how you hit O(1)
  extra space.

Order matters: `res[i] = prefix` **before** updating `prefix`, in both directions. Test
`[0,0]` → `[0,0]` and `[1,0]` → `[0,1]`.

### 8.6 2D prefix sums — rectangles (LC 304)

```js
function build2D(m) {
  const rows = m.length, cols = m[0].length;
  const pre = Array.from({ length: rows + 1 }, () => new Array(cols + 1).fill(0));
  for (let i = 0; i < rows; i++)
    for (let j = 0; j < cols; j++)
      pre[i + 1][j + 1] = m[i][j] + pre[i][j + 1] + pre[i + 1][j] - pre[i][j];
  return pre;
}
// O(rows × cols) build and space.

// Sum of the rectangle from (r1,c1) to (r2,c2), inclusive — O(1) per query:
const rect = (pre, r1, c1, r2, c2) =>
  pre[r2 + 1][c2 + 1] - pre[r1][c2 + 1] - pre[r2 + 1][c1] + pre[r1][c1];
```

Both formulas are the same idea, called inclusion–exclusion: **subtract the strip above,
subtract the strip to the left, then add back the corner you just subtracted twice.**

Draw the four rectangles on the whiteboard while you explain it — this is one of the few
places a diagram genuinely helps, and `+ pre[r1][c1]` is the term people forget.

Same `+1` padding convention as 1D, for the same reason: no special case for row 0 or
column 0.

### 8.7 Difference arrays — the reverse operation

Prefix sums make many range **questions** fast. Difference arrays make many range
**updates** fast.

To add `v` to everything in range `l..r`, mark just two positions, then reconstruct once at
the end:

```js
// Corporate Flight Bookings (LC 1109): bookings = [[first, last, seats], ...], 1-indexed
function corpFlightBookings(bookings, n) {
  const diff = new Array(n + 1).fill(0);
  for (const [first, last, seats] of bookings) {
    diff[first - 1] += seats;
    diff[last] -= seats;                    // one past the end
  }
  for (let i = 1; i < n; i++) diff[i] += diff[i - 1];   // prefix-sum it back
  return diff.slice(0, n);
}
// O(n + number of bookings) time, O(n) space.
// Looping over each booking's whole range would be O(n × bookings).
```

`diff[l] += v; diff[r+1] -= v;` **is** the entire technique. Size the array `n+1` so `r+1`
is never out of bounds when `r === n-1`. **Car Pooling (LC 1094)** is the same problem with
a capacity check.

**This is the one pattern in the phase with a direct, non-contrived frontend use:**
applying hundreds of overlapping range updates — timeline/Gantt overlays, per-day
availability counts, heatmap buckets — without touching every cell for every update.

### 8.8 Mistakes

| Mistake | Result |
|---|---|
| Using `pre[i]` = "sum including i" instead of the `n+1` form | Off-by-one in every query |
| Forgetting the `[0, 1]` seed in LC 560 | Misses every subarray starting at index 0 |
| Inserting into the map before checking | Over-counts when `k === 0` |
| Forgetting `((x % k) + k) % k` | Silent `NaN` on negative numbers |
| Overwriting the map in the "longest" variant | You get the shortest answer instead |
| Updating `prefix` before writing `res[i]` in LC 238 | Each answer includes its own element |
| Building a prefix **array** when a running **variable** would do | O(n) space you didn't need |

---

## Part 9 — BONUS TRICK: Kadane (best chunk sum)
### Use it for "maximum sum subarray", "best profit", "max product subarray"

Not strictly two pointers or a window, but it lives here because the signal — "maximum sum
contiguous subarray" — looks *identical* to a window problem. It's also the bridge into
Phase 10's Dynamic Programming.

### 9.1 What it is

At every element you answer **exactly one question**:

> Is the best chunk *ending right here* just this element alone, or this element glued onto
> the best chunk that ended at the previous position?

```js
function maxSubArray(nums) {
  let cur = nums[0], best = nums[0];         // ⚠️ start from nums[0], NOT from 0
  for (let i = 1; i < nums.length; i++) {
    cur  = Math.max(nums[i], cur + nums[i]); // best chunk ENDING HERE
    best = Math.max(best, cur);              // best chunk ANYWHERE so far
  }
  return best;
}
// O(n) time, O(1) space.
```

Two variables, two different meanings — this is the part people mix up:

- **`cur`** = the best chunk that **ends exactly at the current position**
- **`best`** = the best chunk seen **anywhere** so far

### 9.2 Watch it run — `[-2, 1, -3, 4, -1, 2, 1, -5, 4]`

| `i` | `nums[i]` | `cur + nums[i]` | `cur` = max of the two | `best` |
|---|---|---|---|---|
| 0 | −2 | – | −2 | −2 |
| 1 | 1 | −1 | **1** (restart) | 1 |
| 2 | −3 | −2 | −2 (extend) | 1 |
| 3 | 4 | 2 | **4** (restart) | 4 |
| 4 | −1 | 3 | 3 (extend) | 4 |
| 5 | 2 | 5 | 5 (extend) | 5 |
| 6 | 1 | 6 | 6 (extend) | **6** |
| 7 | −5 | 1 | 1 (extend) | 6 |
| 8 | 4 | 5 | 5 (extend) | 6 |

Answer: **6** (the chunk `[4, -1, 2, 1]`).

Watch rows 1 and 3: when the running total drags you down, `cur` **restarts** from the
current element. That restart *is* the algorithm.

### 9.3 The initialization trap — the thing interviewers test

Starting with `best = 0` returns **0** for `[-3, -1, -2]`. The right answer is **−1**.
Why? `best = 0` silently assumes "an empty subarray is allowed". LC 53 requires at least
one element, so it isn't. Initializing from `nums[0]` handles all-negative arrays for free.

**Now compare Best Time to Buy and Sell Stock (LC 121):**

```js
function maxProfit(prices) {
  let minSoFar = Infinity, best = 0;         // ⭐ best = 0 IS correct here
  for (const p of prices) {
    minSoFar = Math.min(minSoFar, p);
    best = Math.max(best, p - minSoFar);
  }
  return best;
}
// O(n) time, O(1) space.
```

`best = 0` is right here because **doing nothing is explicitly allowed** — you may simply
not trade, and the profit is 0. Same question, opposite answer, decided entirely by
**reading which one the problem permits**. Interviewers test this with an all-decreasing
price array.

It's also Kadane in disguise: the best profit is the maximum-sum subarray of the
day-to-day differences. Mentioning that equivalence is a nice touch and makes the LC 122+
variants easier to reason about.

### 9.4 Returning the indexes too — a common follow-up

```js
function maxSubArrayWithRange(nums) {
  let cur = nums[0], best = nums[0], start = 0, bestL = 0, bestR = 0;
  for (let i = 1; i < nums.length; i++) {
    if (cur + nums[i] < nums[i]) { cur = nums[i]; start = i; }   // restarting → new start
    else cur += nums[i];
    if (cur > best) { best = cur; bestL = start; bestR = i; }    // strict > keeps the FIRST best
  }
  return { sum: best, from: bestL, to: bestR };
}
// O(n) time, O(1) space.
```

Note `cur > best` (strict). With `>=` you'd report the **last** tied range instead of the
first. When a problem says "if there are ties, return the earliest", that comparison
operator *is* the answer.

### 9.5 Maximum Product Subarray (LC 152) — why it needs two variables

Multiplying by a negative flips the smallest into the largest, so tracking only the maximum
isn't enough. Track both:

```js
function maxProduct(nums) {
  let mx = nums[0], mn = nums[0], best = nums[0];
  for (let i = 1; i < nums.length; i++) {
    const x = nums[i];
    const cands = [x, mx * x, mn * x];    // ⭐ compute all three BEFORE reassigning
    mx = Math.max(...cands);
    mn = Math.min(...cands);
    best = Math.max(best, mx);
  }
  return best;
}
// O(n) time, O(1) space. Math.max over a fixed 3-element array is O(1).
```

**The mistake:** assigning `mx` first and then using the *new* `mx` to compute `mn`.
Compute the candidates from the old values, then assign both. Zeros are handled for free —
`x` itself is always a candidate, which restarts the run.

### 9.6 Why not a sliding window?

With negatives, "shrink from the left" isn't well defined: a prefix with a negative sum
should be **dropped entirely**, not shrunk one element at a time. Kadane's
`max(nums[i], cur + nums[i])` *is* that drop, done in O(1).

Notice too: if all values were positive, the answer would trivially be the whole array —
which tells you the problem only exists **because** of the negatives, and therefore isn't a
window problem. Voicing that reasoning shows you *chose* the pattern rather than
pattern-matching the word "subarray".

### 9.7 The shape, and why it matters for Phase 10

Every problem in this Part is:

```js
let cur = <base>, best = <base>;
for (each element) {
  cur  = <extend the run, or restart at this element>;   // best answer ENDING HERE
  best = <better of best and cur>;                       // best answer ANYWHERE
}
```

That distinction — **"best ending here"** vs **"best anywhere"** — is the core of 1D
Dynamic Programming, and you're already writing it in O(1) space. When Phase 10 introduces
`dp[i]` arrays, it's this exact thing with the history kept. Recognizing that now makes
Phase 10 dramatically shorter.

**Say it in DP language in the interview:**
> *"Let `cur` be the maximum sum of a subarray ending exactly at `i`. Then
> `cur[i] = max(nums[i], cur[i-1] + nums[i])`, and the answer is the max over all `i`. I
> only need the previous value, so it's O(1) space instead of an O(n) table."*

That sentence earns you credit in the DP round before you even get there.

---

## Part 10 — BONUS TRICK: Use the array as its own lookup table
### Use it when values are numbers 1..n and the problem demands O(1) space

**Be honest with yourself about this family: these are tricks.** Learn them; don't expect
to derive them live under pressure.

The enabling condition is always the same: **the values are constrained to 1..n (or 0..n)
where n is the array length.** That means value `7` "belongs" at index 6, so the array can
act as its own hash table — no extra memory.

### 10.1 Missing Number (LC 268) — values 0..n, one missing

```js
// Version 1: the sum formula
function missingNumber(nums) {
  const n = nums.length;
  const expected = (n * (n + 1)) / 2;      // sum of 0..n
  let actual = 0;
  for (const x of nums) actual += x;
  return expected - actual;
}
// O(n) time, O(1) space.

// Version 2: XOR — no size concerns at all
function missingNumberXor(nums) {
  let x = nums.length;
  for (let i = 0; i < nums.length; i++) x ^= i ^ nums[i];
  return x;
}
// O(n) time, O(1) space.
```

JS numbers are exact integers up to 2^53, so the sum version can't silently overflow the
way a 32-bit `int` would in Java or C++. Worth one sentence — it's a real cross-language
difference and explains why the XOR variant exists elsewhere. (Note `^` itself coerces to
32-bit signed integers, which is fine here but is its own trap for large values.)

### 10.2 Find All Numbers Disappeared (LC 448) — negation marking

Values are 1..n. Mark "I saw this value" by making the value at the corresponding index
negative:

```js
function findDisappearedNumbers(nums) {
  for (const x of nums) {
    const i = Math.abs(x) - 1;              // ⭐ abs, because it may already be marked
    if (nums[i] > 0) nums[i] = -nums[i];
  }
  const res = [];
  for (let i = 0; i < nums.length; i++) if (nums[i] > 0) res.push(i + 1);
  return res;
}
// O(n) time, O(1) auxiliary space (the output doesn't count).
```

Always read the value through `Math.abs` — after the first few steps some entries are
already negative, and forgetting that gives you a negative index. **Find All Duplicates
(LC 442)** is the same loop, collecting `Math.abs(x)` when the target slot is *already*
negative.

⚠️ **State the caveat:** this **mutates the input**, which is only acceptable if the
problem allows it. If it doesn't, you're back to a `Set` and O(n) space. Volunteering that
trade beats being caught by it.

### 10.3 First Missing Positive (LC 41) — cyclic sort

Put each value `v` (where `1 ≤ v ≤ n`) at index `v−1`, then scan for the first slot that's
wrong:

```js
function firstMissingPositive(nums) {
  const n = nums.length;
  for (let i = 0; i < n; i++) {
    while (nums[i] > 0 && nums[i] <= n && nums[nums[i] - 1] !== nums[i]) {
      const j = nums[i] - 1;
      [nums[i], nums[j]] = [nums[j], nums[i]];
    }
  }
  for (let i = 0; i < n; i++) if (nums[i] !== i + 1) return i + 1;
  return n + 1;
}
// O(n) time, O(1) space.
```

Two defences to have ready:

1. **Why the nested `while` is still O(n):** every swap puts at least one value into its
   final correct slot, and a value never leaves a correct slot. So there are at most n
   swaps *in total* across the whole outer loop. **Same amortized argument as sliding
   window** — the inner counter persists, it doesn't reset.
2. **Why the condition compares values (`nums[nums[i]-1] !== nums[i]`) rather than
   indexes:** the value-based test also terminates on duplicates. With an index-based test,
   `[1, 1]` swaps forever.

Also worth saying: the answer is always in `1..n+1`, which is why values outside that range
can be ignored entirely. Test `[1,2,3]` → 4, `[7,8,9]` → 1, `[3,4,-1,1]` → 2.

### 10.4 Find the Duplicate Number (LC 287) — recognize it, implement it later

Values in `1..n`, exactly one duplicate, **must not modify the array**, O(1) space. The
intended solution treats the array as a linked list (`i → nums[i]`) and uses Floyd's cycle
detection — which belongs to Phase 5. Recognize it now, implement it there.

If you're asked before Phase 5, the binary-search-on-the-value-range answer is O(n log n)
and fully acceptable:

```js
function findDuplicate(nums) {
  let lo = 1, hi = nums.length - 1;
  while (lo < hi) {
    const mid = (lo + hi) >> 1;
    let count = 0;
    for (const x of nums) if (x <= mid) count++;
    if (count > mid) hi = mid;      // too many values ≤ mid → the duplicate is in [lo, mid]
    else lo = mid + 1;
  }
  return lo;
}
// O(n log n) time, O(1) space. This is "binary search on the answer space" — a preview of Phase 3.
```

---
## Part 11 — JavaScript traps
### The things that break your *correct* algorithm because of the language

Reminder from Part 0: **"V8" just means "the program that runs your JavaScript"** —
Google's engine, inside Chrome and Node. A "V8 gotcha" is *a JavaScript quirk that a
Python or Java candidate in the same interview wouldn't hit*. Nothing more.

Phase 0 covered the general cost traps. These are the ones that specifically bite on
array, string and two-pointer problems.

### 1. `Math.max(...arr)` crashes on big arrays

```js
Math.max(...bigArray);   // ❌ RangeError: Maximum call stack size exceeded
```

Spread turns every element into a **function argument**, and V8's argument limit is roughly
65,000–125,000. It works on every small test you run locally and blows up on the real input.

```js
let mx = -Infinity;
for (const x of arr) if (x > mx) mx = x;             // ✅ O(n), no limit
arr.reduce((a, b) => Math.max(a, b), -Infinity);     // ✅ also fine
```

**But** `Math.max(...count)` on a fixed 26-element array (Part 7.10) is perfectly safe and
O(1) — the length is a constant. The rule is only about arrays whose length grows with the
input. Being able to draw that line is more impressive than blanket-avoiding spread.

### 2. `sort()` sorts as text, and mutates the original

```js
[10, 9, 1].sort();                  // [1, 10, 9]   ← lexicographic, i.e. wrong
[10, 9, 1].sort((a, b) => a - b);   // [1, 9, 10]   ✅
```

Descending is `(a, b) => b - a`. **Never** `(a, b) => a > b` — a boolean is not a valid
comparator (it coerces to 0/1 and can never return −1), so it produces subtly wrong orders.

Every k-sum solution in Part 6 mutates the caller's array. Say it:
*"this sorts in place, so it mutates the input — if the caller needs the original order
I'd copy first, which costs O(n) space."* On LeetCode nobody cares; in a code review it's
a bug, and interviewers from product teams notice.

### 3. `%` is remainder, not modulo

```js
-1 % 5;                  // -1   (Python gives 4)
((sum % k) + k) % k      // the only form to write when values can be negative
```

Repeated here because it silently produces `NaN` rather than throwing.

### 4. Out-of-bounds reads give `undefined`, then `NaN`, silently

```js
const arr = [1, 2, 3];
arr[5] + 1;              // NaN — not an error
sum += arr[i];           // one bad read poisons everything downstream
```

Java throws. Python throws. JavaScript hands you `NaN` fifty lines later with no stack
trace. **Fastest debugging heuristic in this phase: if your answer is `NaN`, you have an
out-of-bounds read** (or an unnormalized negative `%`).

### 5. `slice` inside a loop turns O(n) into O(n²)

```js
helper(arr.slice(i));      // ❌ copies the array on every call
helper(arr, i);            // ✅ pass the index instead
s.slice(l, r + 1)          // ❌ inside a window loop — THE classic mistake
```

Pass **indexes**, not copies. Slice once at the very end if you need the actual substring.
This is the top source of accidental O(n²) in this phase.

### 6. `new Array(n)` creates holes, which are slow and behave strangely

```js
new Array(3);                       // [ <3 empty items> ] — holes, not undefined
new Array(3).fill(0);               // ✅ [0, 0, 0]
Array.from({ length: 3 }, () => 0); // ✅ also fine

const holey = [1, , 3];
holey.map(x => x * 2);              // [2, <empty>, 6] — map SKIPS holes
```

Always `.fill()` or `Array.from`. Never `delete arr[i]` — it punches a hole.

### 7. `Array(n).fill([])` gives you n references to the SAME array

```js
const grid = Array(3).fill([]);     // ❌ all three rows are one array
grid[0].push(1);
grid;                               // [[1], [1], [1]]  😱

const ok = Array.from({ length: 3 }, () => []);                         // ✅
const dp = Array.from({ length: rows }, () => new Array(cols).fill(0)); // ✅
```

`fill` with an object fills with **one reference**; `Array.from` with a factory calls it
fresh per index.

### 8. `Object.keys(obj).length` is O(n); `map.size` is O(1)

Inside a window loop, `Object.keys(...).length` turns O(n) into O(n·k). Use a `Map`.

Also, object keys are always strings; Map keys are not:

```js
const o = {}; o[1] = 'a'; o['1'] = 'b';
o;                                        // { '1': 'b' } — ONE key, coerced

const m = new Map(); m.set(1, 'a').set('1', 'b');
m.size;                                   // 2 ✅ distinct keys
```

Use `Map` for window counters whenever keys are numbers, and always for `size`.

### 9. `for...in` on arrays is always wrong

```js
const a = [10, 20, 30];
for (const i in a) console.log(i + 1);   // "01", "11", "21" — the keys are STRINGS
```

It also walks inherited properties. Use an index `for` loop, or `for...of`.

### 10. `arr.length` is re-read every iteration

```js
for (let i = 0; i < arr.length; i++) { arr.push(x); }   // ❌ infinite loop
```

If the loop body changes the length, cache it first: `const n = arr.length`.

### 11. Strings: `str[i]` is O(1), `slice` is O(n), and strings are immutable

```js
s[i]              // O(1) — use this in two-pointer and window loops
s.charAt(i)       // O(1) — same thing, older style
s.charCodeAt(i)   // O(1) — use for the 26-bucket count trick
s.slice(a, b)     // O(n) — NEVER inside a window loop
s.split('')       // O(n) allocation, and it breaks emoji
```

The counting idiom to have in muscle memory:

```js
const idx = c => c.charCodeAt(0) - 97;              // 'a' → 0, lowercase only
const counts = new Array(26).fill(0);
```

If the problem says "letters" without specifying case, **ask**. If it says any ASCII, use
`new Array(128).fill(0)` or a `Map`.

Because strings are immutable, `s[0] = 'x'` silently does nothing, and there is **no
O(1)-space string reversal in JS**.

### 12. Unicode: `length` lies

```js
'👍'.length;             // 2  — one symbol, two UTF-16 units
[...'👍'].length;        // 1  — spread iterates real characters

'ab👍'.split('').reverse().join('');   // mojibake — the emoji is torn in half
[...'ab👍'].reverse().join('');        // '👍ba' ✅
```

For LeetCode-style ASCII input none of this matters. In an interview, one sentence —
*"I'm assuming ASCII; with real user input I'd iterate code points via `[...str]`, or use
`Intl.Segmenter` for grapheme clusters"* — is exactly the kind of frontend-flavoured depth
that lands, because it's a bug they've actually shipped.

### Two smaller ones

**`indexOf` vs `includes` on `NaN`:**

```js
[NaN].indexOf(NaN);     // -1     (strict equality: NaN !== NaN)
[NaN].includes(NaN);    // true   (uses SameValueZero)
[-0].includes(0);       // true
```

Rarely decisive, but if a problem involves `NaN`, `includes` is the one that behaves as
you'd expect. Both are still O(n) — never in a loop.

**No integer overflow, so skip the overflow-safe midpoint ritual:**

```js
Math.floor((lo + hi) / 2);        // ✅ fine in JS for any realistic n
lo + Math.floor((hi - lo) / 2);   // the Java/C++ habit — harmless, unnecessary here
(lo + hi) >> 1;                   // fast, but silently breaks above 2^31
```

Exact integers hold to 2^53, so `lo + hi` can't overflow for any array you could allocate.
If you write the Java form anyway, say you know it's belt-and-braces in JS — otherwise it
reads as copied from another language.

---

## Part 12 — Complexity of every pattern here

| Pattern | Time | Space | Note |
|---|---|---|---|
| Write pointer | O(n) | O(1) | one read pass, ≤ n writes |
| Two pointers, opposite ends | O(n) | O(1) | needs sorted input (or symmetry) |
| Sort **then** two pointers | O(n log n) | O(1)–O(n) | the sort dominates |
| 3Sum | O(n²) | O(1) extra | n outer × O(n) scan |
| 4Sum / k-Sum | O(n^(k−1)) | O(1) extra | (k−2) loops + a scan |
| Fixed sliding window | O(n) | O(1) or O(k) | O(k) only if the state is a map/set |
| Variable sliding window | O(n) | O(1)–O(alphabet) | pointer movement ≤ 2n |
| Window + `Map` counter | O(n) | O(k) | `map.size` is O(1) — use it |
| Window + 26-element array | O(n) | O(1) | a fixed alphabet is a constant |
| Exactly-k via two atMost passes | O(n) | O(k) | two linear passes |
| Prefix sum array | O(n) build / O(1) query | O(n) | worth it for many queries |
| Prefix sum, streamed | O(n) | O(1) | when you need only "so far" |
| Prefix sum + hash map | O(n) | O(n) | **handles negatives; windows don't** |
| 2D prefix sum | O(mn) build / O(1) query | O(mn) | inclusion–exclusion |
| Difference array | O(n + updates) | O(n) | many range updates |
| Kadane / running value | O(n) | O(1) | one-state DP |
| Negation marking | O(n) | O(1) extra | mutates the input |
| Cyclic sort | O(n) | O(1) | ≤ n swaps in total |
| Brute force, every pair | O(n²) | O(1) | the baseline you name, then beat |
| Brute force, every subarray | O(n²) or O(n³) | O(1) | O(n³) if you re-sum each one |

**Two lines worth internalizing:**

- **There are n(n+1)/2 subarrays, so checking all of them is O(n²)** — and O(n³) if you
  re-sum each from scratch. Opening with *"brute force is O(n³), O(n²) with a running sum,
  O(n) with a window"* is a strong three-step ladder for any subarray problem.
- **Sorting to enable two pointers costs O(n log n)** — so if a hash map gets you O(n), the
  hash map wins. **Sort when you need ordering** (dedup, triplets, ranges). **Hash when you
  need lookup.**

---

## Part 13 — Where this actually shows up in frontend work
### The honest version — some of these connections are real, some you'll see claimed online are forced

**Sliding window → genuinely everywhere in frontend.**
- **Rate limiting and analytics:** "how many events in the last 60 seconds" is a variable
  window over a timestamp array. That's exactly how client-side throttling of telemetry
  batches is built.
- **Moving averages for smoothing:** FPS counters, scroll velocity, pointer/sensor data. A
  fixed window with `sum += arr[i] - arr[i-k]` *is* the implementation.
- **Virtualized lists:** the set of rendered rows **is** a window over the data array, and
  `startIndex`/`endIndex` are `l` and `r`. react-window and TanStack Virtual maintain
  exactly this.

**Prefix sums → variable-height virtualization.** The strongest real example in the phase.
To render a virtualized list where rows have different heights, you keep a prefix sum of
heights; finding the first visible row for a given `scrollTop` is a **binary search over
that prefix array** — O(log n) per scroll event instead of O(n). Building it is O(n) once;
changing one row's height invalidates the suffix. If you've used TanStack Virtual's dynamic
measurement, that's the data structure underneath.

**Difference arrays → overlapping range updates.** Availability calendars, Gantt/timeline
overlays, "how many bookings overlap on each day". Applying `u` updates naively is O(n·u);
the difference array makes it O(n + u).

**Two pointers → merging two sorted streams.** Reconciling a sorted server page with a
sorted local cache, diffing two sorted ID lists to compute added/removed sets, merging
sorted results from two endpoints. This is a real thing you write in a data layer, and it's
why `merge` in Part 5.9 is worth knowing cold.

**Write pointer → mostly interview-only in React, and you should say so.** In React you
*don't* mutate state arrays; you produce new ones with `filter`/`map`, because reference
identity drives re-renders. The in-place write pointer is the wrong tool inside a
component. Where it **is** right: hot non-React code — canvas/WebGL particle buffers, audio
processing, large data transforms in a worker where allocation pressure and GC pauses
matter.

> If an interviewer asks *"would you write this in your React app?"*, the correct answer is
> **"no — immutability wins there; this matters in a worker or a canvas loop where
> allocations cause frame drops."** That answer is better than pretending the trick is
> universally applicable.

**The DOM angle** for `moveZeroesSwap`'s `if (w !== r)` guard: skipping no-op swaps matters
concretely when the "swap" is `insertBefore` on real nodes, because every unnecessary DOM
move costs layout work.

**Kadane → the weakest link.** Occasionally useful for "best streak" metrics in a
dashboard. Don't oversell it; the honest pitch is that it's the O(1)-space DP pattern that
Phase 10 builds on.

---

## Part 14 — What to actually say out loud

Say the **reason**, not just the name. "Two pointers" is a label. *"Sortedness lets me
discard a whole side per comparison"* is understanding.

**Opening any array problem:**
> "Brute force is checking every pair, O(n²). But the array is sorted — that means
> comparing the two ends tells me which end can't be part of the answer, so I can converge
> two pointers in O(n) with O(1) space. Want me to go straight there, or start with brute
> force and improve it?"

**Write pointer:**
> "I'll keep a write index trailing a read index. The invariant is that `arr[0..w-1]` is
> always the final answer so far, and `w ≤ r` always, so I never overwrite anything I still
> need. O(n) time, O(1) space, and it returns the new length as asked."

**Two pointers, opposite ends:**
> "I'm moving the pointer that can't improve the answer. `nums[l] + nums[r]` is below the
> target, and everything left of `r` is smaller, so `nums[l]` can't pair with anything in
> range — `l` is done permanently. Each pointer only moves inward, at most n moves total,
> so O(n)."

**Container With Most Water specifically:**
> "The area is capped by the shorter wall, so any other container using that wall has less
> width and no more height — it can never beat what I just recorded. That's what makes
> discarding the shorter side safe."

**Sliding window — have this word-perfect:**
> "This is a contiguous-subarray problem and validity is monotone: adding on the right can
> only make it worse, removing from the left can only make it better. So I'll expand right
> every step and shrink left while the window is invalid, recording the best after each
> shrink. It looks like a nested loop but it's O(n) — `left` never resets, it only moves
> forward, at most n times across the whole run. Total pointer movement is bounded by 2n."

**When they challenge the nested loop:**
> "The inner `while` doesn't multiply, it adds. If the inner counter reset each outer
> iteration it'd be O(n²); it persists, so total inner work is bounded by n across all
> iterations. Amortized O(1) per step."

**Showing you know the pattern's limits — high value, most candidates never say it:**
> "One caveat: this relies on all values being positive. With negatives, adding an element
> can decrease the sum, so shrinking is no longer guaranteed to help and the window breaks.
> I'd switch to prefix sums with a hash map, or a monotonic deque if I needed the shortest
> such subarray."

**Prefix sum:**
> "A range sum is the difference of two running totals, so I'll keep a running sum and a map
> from prefix value to how many times I've seen it. A subarray sums to k exactly when some
> earlier prefix equals `sum − k`. O(n) time, O(n) space — and unlike a sliding window this
> handles negative numbers."

**Kadane:**
> "One-state DP: `cur` is the best subarray sum ending exactly here — either this element
> alone, or this element extending the previous run. The answer is the max over all
> positions. I only need the previous value, so O(1) space instead of an O(n) table."

**Closing any solution — always volunteer space, always mention mutation:**
> "Time is O(n), space is O(1) auxiliary. One note: this sorts/mutates the input array — if
> the caller needs it preserved I'd copy first, which makes it O(n) space."

**When you're stuck — narrate the search, never go silent:**
> "Let me check what structure I can exploit. It's not sorted, so opposite-end pointers are
> out unless I pay O(n log n) to sort — and sorting destroys the indexes, which the output
> needs. The answer is contiguous, so a window is plausible, but there are negative numbers,
> so validity isn't monotone. That points at prefix sums plus a hash map."

**That last one is the most valuable thing on this page.** Interviewers hire the person who
narrates a structured search over the person who silently produces a memorized answer.

---

## Part 15 — Before you say "I think that's right"

Every failure in this phase is one of these six. Check them **in this order**.

1. **Window length** — it's `r - l + 1`, not `r - l`. Verify with `l === r`: one element,
   length must be 1.
2. **Loop bound** — windows of size k by start index: `i + k <= n`. Adjacent pairs:
   `i < n - 1`. Last element: `i <= n - 1`.
3. **Which element leaves** — in a fixed window at step `i`, the leaver is `arr[i - k]`.
4. **Where you record the answer** — LONGEST records *after* the shrink loop, SHORTEST
   records *inside* it. Backwards passes the sample test and fails the hidden ones.
5. **Initialization** — `best = 0` when an empty answer is legal; `best = nums[0]` or
   `-Infinity` when it isn't; `best = Infinity` when minimizing.
6. **Strictness** — `l < r` for distinct pairs, `l <= r` when the middle element must be
   processed. `>` keeps the **first** tie, `>=` keeps the **last**.

### The dry-run protocol — 90 seconds that catches nearly everything

**Before you run any code**, trace these five inputs on paper:

| Input | What it catches |
|---|---|
| `[]` | boundary reads, `length - 1` going negative, `reduce` with no initial value |
| `[5]` | `l < r` never entering the loop, loops starting at `i = 1` never running |
| `[2, 2]` or `"aa"` | duplicate handling, shrink-before-add ordering |
| all negatives, e.g. `[-3, -1, -2]` | initialization traps (Kadane, min/max seeds) |
| the smallest input where the answer is **not** the whole array | recording in the wrong place |

Write the trace as a table — `l`, `r`, state, `best`, one row per step, exactly like the
tables in this document. Interviewers explicitly value seeing this, and Phase 0's rule
stands: **debugging by clicking Run is a red flag in a live interview.**

### Two instant diagnoses

- **Answer is `NaN`** → an out-of-bounds read, or an unnormalized negative `%`.
- **Answer is exactly the array length, or exactly 0, when it shouldn't be** → your
  recording position or your initialization is wrong, not your logic.

---

## Part 16 — Your 7-day schedule (Days 3–9)

**Guided** = you already know which trick applies. **Blind** = you have to work it out
yourself. All numbers are LeetCode problem numbers.

### Day 3 — Traversal, in place, write pointer
Read: Parts 1, 2, 3, 4.

| | Problems |
|---|---|
| Guided | 27 Remove Element · 26 Remove Duplicates from Sorted Array |
| Blind | 283 Move Zeroes · 80 Remove Duplicates from Sorted Array II |
| Stretch | 88 Merge Sorted Array (backward fill) |

**Target:** write the write-pointer template from memory and state both invariants out loud.

### Day 4 — Two pointers, opposite ends
Read: Part 5.

| | Problems |
|---|---|
| Guided | 344 Reverse String · 167 Two Sum II |
| Blind | 125 Valid Palindrome · 11 Container With Most Water |
| Stretch | 42 Trapping Rain Water (write the O(n)-space version first, then compress) |

**Target:** give the Container With Most Water exchange argument unprompted.

### Day 5 — Sort + two pointers (the hardest day)
Read: Part 6.

| | Problems |
|---|---|
| Guided | 977 Squares of a Sorted Array · 75 Sort Colors |
| Blind | 15 3Sum · 16 3Sum Closest |
| Stretch | 18 4Sum · 259 3Sum Smaller |

**Target:** 3Sum with all three duplicate skips correct, from memory. If you only half-get
3Sum, redo it on Day 9 rather than moving on with a shaky version.

### Day 6 — Fixed sliding window
Read: Parts 7.1–7.5.

| | Problems |
|---|---|
| Guided | 643 Maximum Average Subarray I · "max sum subarray of size k" (classic, not on LC) |
| Blind | 438 Find All Anagrams in a String · 567 Permutation in String |
| Stretch | 1456 Maximum Number of Vowels in a Substring of Given Length |

**Target:** `sum += arr[i] - arr[i-k]` without hesitating, and explain the `i >= k - 1`
recording gate.

### Day 7 — Variable sliding window (the highest-value day in the phase)
Read: Parts 7.6–7.15.

| | Problems |
|---|---|
| Guided | 3 Longest Substring Without Repeating Characters · 209 Minimum Size Subarray Sum |
| Blind | 424 Longest Repeating Character Replacement · 1004 Max Consecutive Ones III · 904 Fruit Into Baskets |
| Stretch | 76 Minimum Window Substring · 713 Subarray Product Less Than K |

**Target:** write both Template B and Template C from memory, and decide which one a new
problem needs within 15 seconds of reading it.

### Day 8 — Prefix sums and running values
Read: Parts 8 and 9.

| | Problems |
|---|---|
| Guided | 303 Range Sum Query Immutable · 560 Subarray Sum Equals K |
| Blind | 238 Product of Array Except Self · 53 Maximum Subarray · 525 Contiguous Array |
| Stretch | 974 Subarray Sums Divisible by K · 304 Range Sum Query 2D · 1109 Corporate Flight Bookings |

**Target:** explain why 560 needs prefix sums instead of a sliding window, and write
`((x % k) + k) % k` without prompting.

### Day 9 — Mixed practice, no hints
Read: Parts 2, 14, 15, and the cheat sheet.

Work the Part 17 practice set cold. For each problem, **before writing anything**: name the
trick, state the target time and space, then write the skeleton.

| | Problems |
|---|---|
| Mixed blind | 121 Best Time to Buy and Sell Stock · 152 Maximum Product Subarray · 448 Find All Numbers Disappeared · 992 Subarrays with K Different Integers |
| Re-solve | Anything from Days 3–8 you needed help with — and the 3-day/10-day re-solve rule starts now |

**Target:** 60-second trick identification on unseen problems, with the target complexity
stated before any code.

---
## Part 17 — Practice: 32 problems

Answers are in Part 18. Work these cover-down.

**Protocol for each one, in this order:**

1. Name the trick (from the table in Part 2).
2. State the target time and space **before** writing code.
3. Write the skeleton from memory.
4. Dry-run the five inputs from Part 15.
5. Only then check the answer.

Problems **1–12 are deliberate recall** — they're solved in the body above, so if you can't
reproduce them cold, the body didn't stick. Problems **13–32 are variants that are not
worked above**; they're the real test of whether you own the patterns or merely recognize
them.

### Recall set (solved above — reproduce from memory)

**1.** (LC 80) Remove duplicates from a sorted array so each value appears **at most
twice**, in place. Return the new length. Then generalize to at most `k`.

**2.** (LC 283) Move all zeroes to the end in place, preserving the order of the
non-zeroes. Minimize the number of writes.

**3.** (LC 88) `nums1` has `m` sorted values followed by `n` empty slots; `nums2` has `n`
sorted values. Merge into `nums1` in place, O(1) extra space.

**4.** (LC 125) Is a string a palindrome, considering only alphanumeric characters,
case-insensitive? **O(1) extra space** — no cleaned copy.

**5.** (LC 167) Given a **sorted** array and a target, return the 1-indexed positions of the
two numbers that sum to the target. O(1) space.

**6.** (LC 11) Given `height[]`, pick two lines forming a container holding the most water.
Return the max area, and give the argument for why your pointer move is safe.

**7.** (LC 75) Sort an array of only 0s, 1s and 2s in place, **one pass**, O(1) space.

**8.** (LC 15) Return all **unique** triplets summing to zero.

**9.** (LC 3) Length of the longest substring with no repeating characters.

**10.** (LC 209) Minimal length of a contiguous subarray with sum ≥ target. Return 0 if
none. All values positive.

**11.** (LC 560) Count the subarrays summing to exactly `k`. **Values may be negative.**

**12.** (LC 238) Return an array where `res[i]` is the product of every element except
`nums[i]`. No division, O(1) extra space beyond the output.

### Variant set (not solved above)

**13.** (LC 345) Reverse only the vowels of a string, leaving everything else in place.
`"leetcode"` → `"leotcede"`.

**14.** (LC 392) Is `s` a subsequence of `t`? Follow-up: what changes if you get 10⁹
different values of `s` against one fixed `t`?

**15.** (LC 844) Compare two strings where `#` means backspace. `"ab#c"` and `"ad#c"` are
equal. Do it in **O(1) space** — no string building.

**16.** (LC 680) Can a string be made a palindrome by deleting **at most one** character?

**17.** (LC 881) Each boat carries at most two people and at most `limit` total weight.
Given `people[]`, return the minimum number of boats. (Everyone's weight ≤ limit.)

**18.** (LC 581) Find the length of the shortest continuous subarray that, if sorted, makes
the whole array sorted. Target O(n) time, O(1) space.

**19.** (LC 845) Length of the longest "mountain" — a strictly increasing run followed by a
strictly decreasing run, both non-empty. O(1) space.

**20.** (LC 1299) Replace every element with the greatest element to its right; the last
becomes −1. In place.

**21.** Classic (not on LC): given an array and `k`, return the **maximum sum of any
contiguous subarray of size exactly k**. Handle `k > n`.

**22.** (LC 1456) Maximum number of vowels in any substring of length `k`.

**23.** (LC 1343) Count the subarrays of size `k` whose **average** is ≥ `threshold`. Avoid
floating-point division.

**24.** (LC 2090) For each index, return the average of the subarray of radius `k` centred
there (window size `2k+1`), or −1 where the full window doesn't fit. Integer division,
truncated.

**25.** (LC 1052) `customers[i]` arrive at minute `i`; `grumpy[i]` is 1 if the owner is
grumpy then (those customers leave unsatisfied). The owner can be non-grumpy for one window
of `minutes` consecutive minutes. Return the max satisfied customers.

**26.** (LC 904) You can carry only two types of fruit. Walking left to right through
`fruits[]`, return the maximum you can collect from one contiguous run. (Generalize: at
most `k` distinct.)

**27.** (LC 1004) Given a binary array and `k`, return the longest run of 1s if you may flip
at most `k` zeroes.

**28.** (LC 76) Minimum window substring of `s` containing all characters of `t`, including
duplicates. Return `""` if none.

**29.** (LC 713) Count the contiguous subarrays whose product is strictly less than `k`. All
values ≥ 1.

**30.** (LC 795) Count the contiguous subarrays whose **maximum** element is in
`[left, right]`.

**31.** (LC 696) Count the substrings with equal numbers of consecutive 0s and 1s, where all
the 0s and all the 1s are grouped (`"00110011"` → 6). ⚠️ Careful: this looks like a window
problem and isn't.

**32.** (LC 41) Find the smallest missing **positive** integer. O(n) time, O(1) space.

### Self-check before you look at the answers

- **Which of these are *not* window problems**, despite mentioning subarrays or substrings?
  (Answer: 11, 12, 18, 20, 31, 32 — and 30 only partially. Knowing *why* matters more than
  solving them.)
- **Which three need a sort first, and what does that cost you?** (8, 17, and any k-sum —
  O(n log n), plus loss of the original indexes.)
- **Which mutate their input, and would you flag that in a code review?** (1, 2, 3, 7, 20,
  32, plus 8 and 17 via `sort`.)

---

## Part 18 — Worked answers

### Recall set

**1. Remove Duplicates ≤ k (LC 80)** — write pointer, comparing against the output.

```js
function removeDuplicates(nums, k = 2) {
  let w = 0;
  for (const x of nums) if (w < k || x !== nums[w - k]) nums[w++] = x;
  return w;
}
// O(n) time, O(1) space.
```
**Check:** `nums[w-k]` looks back into the *output*, not the input. If you wrote `nums[r-k]`
you got lucky on some tests and wrong on `[1,1,1,2,2,3]`.

**2. Move Zeroes, minimum writes (LC 283)** — write pointer + swap.

```js
function moveZeroes(nums) {
  let w = 0;
  for (let r = 0; r < nums.length; r++) {
    if (nums[r] !== 0) {
      if (w !== r) [nums[w], nums[r]] = [nums[r], nums[w]];
      w++;
    }
  }
}
// O(n) time, O(1) space.
```
**Check:** the `w !== r` guard. Without it you do n pointless self-swaps — correct output,
unnecessary writes, and the interviewer asked for minimum writes.

**3. Merge Sorted Array (LC 88)** — two pointers, backward fill.

```js
function merge(nums1, m, nums2, n) {
  let i = m - 1, j = n - 1, w = m + n - 1;
  while (j >= 0) nums1[w--] = (i >= 0 && nums1[i] > nums2[j]) ? nums1[i--] : nums2[j--];
}
// O(m + n) time, O(1) space.
```
**Check:** loop on `j >= 0` only. Leftover `nums1` values are already in position, so
copying them is wasted work.

**4. Valid Palindrome, O(1) space (LC 125)** — opposite ends with skips.

```js
function isPalindrome(s) {
  const ok = c => /[a-z0-9]/i.test(c);
  let l = 0, r = s.length - 1;
  while (l < r) {
    while (l < r && !ok(s[l])) l++;
    while (l < r && !ok(s[r])) r--;
    if (s[l].toLowerCase() !== s[r].toLowerCase()) return false;
    l++; r--;
  }
  return true;
}
// O(n) time, O(1) space.
```
**Check:** `l < r` inside **both** inner loops. Test `",."` and `"a."`.

**5. Two Sum II (LC 167)** — opposite ends on sorted input.

```js
function twoSum(nums, target) {
  let l = 0, r = nums.length - 1;
  while (l < r) {
    const s = nums[l] + nums[r];
    if (s === target) return [l + 1, r + 1];
    s < target ? l++ : r--;
  }
  return [];
}
// O(n) time, O(1) space.
```
**Check:** can you say *why* discarding is safe? *"`nums[l]` paired with anything ≤
`nums[r]` is still too small, so `l` is eliminated permanently."*

**6. Container With Most Water (LC 11)** — opposite ends, move the shorter wall.

```js
function maxArea(height) {
  let l = 0, r = height.length - 1, best = 0;
  while (l < r) {
    best = Math.max(best, (r - l) * Math.min(height[l], height[r]));
    height[l] < height[r] ? l++ : r--;
  }
  return best;
}
// O(n) time, O(1) space.
```
**Check:** the exchange argument, in one sentence, without prompting.

**7. Sort Colors (LC 75)** — Dutch national flag.

```js
function sortColors(nums) {
  let low = 0, mid = 0, high = nums.length - 1;
  while (mid <= high) {
    if (nums[mid] === 0) { [nums[low], nums[mid]] = [nums[mid], nums[low]]; low++; mid++; }
    else if (nums[mid] === 1) mid++;
    else { [nums[mid], nums[high]] = [nums[high], nums[mid]]; high--; }
  }
}
// O(n) time, O(1) space, one pass.
```
**Check:** no `mid++` in the `=== 2` branch, and `mid <= high` not `mid < high`. Test
`[2,0,1]` and `[2,2]`.

**8. 3Sum (LC 15)** — sort, fix one, two-pointer the rest.

```js
function threeSum(nums) {
  nums.sort((a, b) => a - b);
  const res = [], n = nums.length;
  for (let i = 0; i < n - 2; i++) {
    if (nums[i] > 0) break;
    if (i > 0 && nums[i] === nums[i - 1]) continue;
    let l = i + 1, r = n - 1;
    while (l < r) {
      const s = nums[i] + nums[l] + nums[r];
      if (s < 0) l++;
      else if (s > 0) r--;
      else {
        res.push([nums[i], nums[l], nums[r]]);
        while (l < r && nums[l] === nums[l + 1]) l++;
        while (l < r && nums[r] === nums[r - 1]) r--;
        l++; r--;
      }
    }
  }
  return res;
}
// O(n²) time, O(1) auxiliary space.
```
**Check:** three skips; `i` compared **backward** to `i-1`; the `l`/`r` skips only inside
the hit branch. Test `[0,0,0,0]` → exactly one triplet.

**9. Longest Substring Without Repeating Characters (LC 3)** — Template B with a `Set`.

```js
function lengthOfLongestSubstring(s) {
  const seen = new Set();
  let l = 0, best = 0;
  for (let r = 0; r < s.length; r++) {
    while (seen.has(s[r])) { seen.delete(s[l]); l++; }
    seen.add(s[r]);
    best = Math.max(best, r - l + 1);
  }
  return best;
}
// O(n) time, O(min(n, alphabet)) space.
```
**Check:** shrink **before** adding. Test `"aa"` → 1 and `"abba"` → 2.

**10. Minimum Size Subarray Sum (LC 209)** — Template C.

```js
function minSubArrayLen(target, nums) {
  let l = 0, sum = 0, best = Infinity;
  for (let r = 0; r < nums.length; r++) {
    sum += nums[r];
    while (sum >= target) { best = Math.min(best, r - l + 1); sum -= nums[l]; l++; }
  }
  return best === Infinity ? 0 : best;
}
// O(n) time, O(1) space.
```
**Check:** record **inside** the shrink loop (minimizing), and convert `Infinity → 0`. Say
the all-positive precondition out loud.

**11. Subarray Sum Equals K (LC 560)** — prefix sum + hash map, because negatives rule out a
window.

```js
function subarraySum(nums, k) {
  const seen = new Map([[0, 1]]);
  let sum = 0, count = 0;
  for (const x of nums) {
    sum += x;
    count += seen.get(sum - k) || 0;
    seen.set(sum, (seen.get(sum) || 0) + 1);
  }
  return count;
}
// O(n) time, O(n) space.
```
**Check:** seed `[0,1]`, and query **before** insert. Test `[1,-1,0]` with `k=0` → 3.

**12. Product of Array Except Self (LC 238)** — prefix and suffix products in the output.

```js
function productExceptSelf(nums) {
  const n = nums.length, res = new Array(n).fill(1);
  let p = 1;
  for (let i = 0; i < n; i++) { res[i] = p; p *= nums[i]; }
  let s = 1;
  for (let i = n - 1; i >= 0; i--) { res[i] *= s; s *= nums[i]; }
  return res;
}
// O(n) time, O(1) auxiliary space (output excluded — say this explicitly).
```
**Check:** `res[i] = p` **before** updating `p`, in both directions. Test `[0,0]` → `[0,0]`
and `[1,0]` → `[0,1]`.

### Variant set

**13. Reverse Vowels (LC 345)** — opposite ends, skip non-targets, swap the rest.

```js
function reverseVowels(s) {
  const isV = c => 'aeiouAEIOU'.includes(c);   // 10-char scan = O(1)
  const a = [...s];                             // strings are immutable → O(n) space is unavoidable
  let l = 0, r = a.length - 1;
  while (l < r) {
    while (l < r && !isV(a[l])) l++;
    while (l < r && !isV(a[r])) r--;
    if (l < r) { [a[l], a[r]] = [a[r], a[l]]; l++; r--; }
  }
  return a.join('');
}
// O(n) time, O(n) space.
```
Volunteer the language point: in C++ this would be O(1) space; in JS you must materialize an
array. That's a real language difference, not a weakness in your solution.

**14. Is Subsequence (LC 392)** — same-direction pointers across two sequences.

```js
function isSubsequence(s, t) {
  let i = 0;
  for (const c of t) if (i < s.length && c === s[i]) i++;
  return i === s.length;
}
// O(|t|) time, O(1) space.
```
**The follow-up is the actual interview question.** For many different `s` against one fixed
`t`: preprocess `t` once into `Map<char, sortedIndexes[]>` in O(|t|), then per query
binary-search the next occurrence after your current position — O(|s| log |t|) per query.
That shift, **from scanning to indexing**, is what they're testing, and it previews Phase 3.

**15. Backspace String Compare (LC 844)** — pointers from the back, counting pending deletes.

```js
function backspaceCompare(s, t) {
  let i = s.length - 1, j = t.length - 1;
  while (true) {
    let skip = 0;
    while (i >= 0 && (s[i] === '#' || skip > 0)) { s[i] === '#' ? skip++ : skip--; i--; }
    skip = 0;
    while (j >= 0 && (t[j] === '#' || skip > 0)) { t[j] === '#' ? skip++ : skip--; j--; }
    if (i < 0 || j < 0) return i < 0 && j < 0;   // both exhausted together → equal
    if (s[i] !== t[j]) return false;
    i--; j--;
  }
}
// O(n + m) time, O(1) space.
```
**Why backward:** a `#` affects the character *before* it, so scanning right-to-left lets you
resolve deletions without a buffer. The O(n)-space version (build both strings with an array,
then compare) is easier — write that first if you're shaky, then compress. Test `"a##c"` vs
`"#a#c"` → true, and `"ab##"` vs `"c#d#"` → true.

**16. Valid Palindrome II (LC 680)** — pointers plus one branching retry.

```js
function validPalindrome(s) {
  const isPal = (l, r) => {
    while (l < r) { if (s[l] !== s[r]) return false; l++; r--; }
    return true;
  };
  let l = 0, r = s.length - 1;
  while (l < r) {
    if (s[l] !== s[r]) return isPal(l + 1, r) || isPal(l, r - 1);   // delete left OR right
    l++; r--;
  }
  return true;
}
// O(n) time, O(1) space.
```
Still O(n), **not** O(n²): the branch happens at most once, and each helper call scans a
disjoint remainder. Say that — it's the follow-up question. Extending to "at most k
deletions" is no longer a two-pointer problem; it becomes edit-distance DP (Phase 10), and
recognizing that boundary is the real follow-up.

**17. Boats to Save People (LC 881)** — sort, then greedy opposite-end pointers.

```js
function numRescueBoats(people, limit) {
  people.sort((a, b) => a - b);
  let l = 0, r = people.length - 1, boats = 0;
  while (l <= r) {                             // ⚠️ <= : the last person still needs a boat
    if (people[l] + people[r] <= limit) l++;   // pair the lightest with the heaviest
    r--;                                       // the heaviest always boards, alone or paired
    boats++;
  }
  return boats;
}
// O(n log n) time (the sort dominates), O(1) auxiliary space.
```
Two things. **`l <= r`, not `l < r`** — this is the one problem in the set where a single
middle element must still be processed, because a lone person occupies a boat. And the greedy
needs justification: *the heaviest person must board some boat, and the best possible
companion is the lightest remaining — if even that doesn't fit, nobody fits, so they go
alone.*

**18. Shortest Unsorted Continuous Subarray (LC 581)** — two directional scans tracking
running extremes.

```js
function findUnsortedSubarray(nums) {
  const n = nums.length;
  let end = -1, start = 0, max = -Infinity, min = Infinity;
  for (let i = 0; i < n; i++) { max = Math.max(max, nums[i]); if (nums[i] < max) end = i; }
  for (let i = n - 1; i >= 0; i--) { min = Math.min(min, nums[i]); if (nums[i] > min) start = i; }
  return end === -1 ? 0 : end - start + 1;
}
// O(n) time, O(1) space.
```
The idea: the **last** index that is smaller than some earlier element must be inside the
window, and symmetrically from the right. The obvious solution — sort a copy and compare — is
O(n log n) time and O(n) space; present that first, then this. `end === -1` means already
sorted → return 0.

**19. Longest Mountain in Array (LC 845)** — one scan consuming up-runs and down-runs.

```js
function longestMountain(arr) {
  const n = arr.length;
  let best = 0, i = 1;
  while (i < n) {
    while (i < n && arr[i] === arr[i - 1]) i++;       // skip flat stretches
    let up = 0;
    while (i < n && arr[i] > arr[i - 1]) { up++; i++; }
    let down = 0;
    while (i < n && arr[i] < arr[i - 1]) { down++; i++; }
    if (up > 0 && down > 0) best = Math.max(best, up + down + 1);
  }
  return best;
}
// O(n) time, O(1) space.
```
Both runs must be non-empty — that's what "mountain" means, and the `up > 0 && down > 0`
guard is the whole problem. The nested `while`s are still O(n) total because `i` never resets:
**the same amortized argument as sliding window.** Test `[2,2,2]` → 0 and `[0,1,0]` → 3.

**20. Replace Elements with Greatest Element on Right (LC 1299)** — suffix maximum, right to
left.

```js
function replaceElements(arr) {
  let mx = -1;
  for (let i = arr.length - 1; i >= 0; i--) {
    const cur = arr[i];
    arr[i] = mx;
    mx = Math.max(mx, cur);
  }
  return arr;
}
// O(n) time, O(1) space.
```
Read the old value into a temp **before** overwriting. This is the "suffix aggregate" half of
Product Except Self — same right-to-left shape, different operator. Brute force is O(n²);
naming the suffix scan is the point.

**21. Max sum subarray of size k (classic)** — fixed window, two-phase form.

```js
function maxSumSizeK(nums, k) {
  if (k > nums.length || k <= 0) return 0;
  let sum = 0;
  for (let i = 0; i < k; i++) sum += nums[i];
  let best = sum;
  for (let i = k; i < nums.length; i++) {
    sum += nums[i] - nums[i - k];
    best = Math.max(best, sum);
  }
  return best;
}
// O(n) time, O(1) space.
```
Everything on Day 6 is built on this. Recomputing each window's sum would be O(n·k). Guard
`k > n` explicitly rather than returning `-Infinity`.

**22. Maximum Number of Vowels in a Substring of Length k (LC 1456)** — fixed window with a
counter.

```js
function maxVowels(s, k) {
  const isV = c => c === 'a' || c === 'e' || c === 'i' || c === 'o' || c === 'u';
  let count = 0;
  for (let i = 0; i < k; i++) if (isV(s[i])) count++;
  let best = count;
  for (let i = k; i < s.length; i++) {
    if (isV(s[i])) count++;
    if (isV(s[i - k])) count--;
    best = Math.max(best, count);
    if (best === k) return k;              // a full window can't be beaten — early exit
  }
  return best;
}
// O(n) time, O(1) space.
```
The early return is a free speed win worth mentioning. Explicit `||` comparisons beat
`'aeiou'.includes(c)` in a hot loop — both are O(1), so say it's a constant-factor choice,
not a complexity one.

**23. Subarrays of Size k with Average ≥ Threshold (LC 1343)** — compare against
`k * threshold`.

```js
function numOfSubarrays(arr, k, threshold) {
  const need = k * threshold;         // ⭐ multiply once instead of dividing per window
  let sum = 0;
  for (let i = 0; i < k; i++) sum += arr[i];
  let count = sum >= need ? 1 : 0;
  for (let i = k; i < arr.length; i++) {
    sum += arr[i] - arr[i - k];
    if (sum >= need) count++;
  }
  return count;
}
// O(n) time, O(1) space.
```
`sum >= k * threshold` instead of `sum / k >= threshold` removes n divisions and all
floating-point error. Same idea as deferring the division in LC 643.

**24. K-Radius Subarray Averages (LC 2090)** — fixed window of size `2k+1`, centred.

```js
function getAverages(nums, k) {
  const n = nums.length, size = 2 * k + 1, res = new Array(n).fill(-1);
  if (size > n) return res;
  let sum = 0;
  for (let i = 0; i < size; i++) sum += nums[i];
  res[k] = Math.floor(sum / size);
  for (let i = size; i < n; i++) {
    sum += nums[i] - nums[i - size];
    res[i - k] = Math.floor(sum / size);      // ⭐ the CENTRE of the window ending at i
  }
  return res;
}
// O(n) time, O(n) output, O(1) auxiliary.
```
Three traps: the window size is `2k+1` (not `2k`); the result index is the window's **centre**
(`i - k`); and `k = 0` must return the array itself (`size = 1` — it does, for free). One
JS-specific line worth saying: sums here reach ~10¹⁰, which is exact in a JS number (safe to
2⁵³), so unlike Java there's no overflow concern and no `long` needed.

**25. Grumpy Bookstore Owner (LC 1052)** — fixed window over the *recoverable* amount.

```js
function maxSatisfied(customers, grumpy, minutes) {
  let base = 0;
  for (let i = 0; i < customers.length; i++) if (grumpy[i] === 0) base += customers[i];

  let gain = 0;
  for (let i = 0; i < minutes; i++) if (grumpy[i] === 1) gain += customers[i];

  let best = gain;
  for (let i = minutes; i < customers.length; i++) {
    if (grumpy[i] === 1) gain += customers[i];
    if (grumpy[i - minutes] === 1) gain -= customers[i - minutes];
    best = Math.max(best, gain);
  }
  return base + best;
}
// O(n) time, O(1) space.
```
**The reframing is the whole problem:** already-satisfied customers are a constant, so the
window should maximize only the *extra* customers the technique saves. Sliding a window over
"total customers" gives a wrong answer that looks reasonable. When a problem has a fixed
baseline plus a windowed bonus, **separate them explicitly** — that habit generalizes.

**26. Fruit Into Baskets / at most k distinct (LC 904)** — Template B with a count map.

```js
function totalFruit(fruits, k = 2) {
  const count = new Map();
  let l = 0, best = 0;
  for (let r = 0; r < fruits.length; r++) {
    count.set(fruits[r], (count.get(fruits[r]) || 0) + 1);
    while (count.size > k) {
      const c = count.get(fruits[l]) - 1;
      if (c === 0) count.delete(fruits[l]); else count.set(fruits[l], c);
      l++;
    }
    best = Math.max(best, r - l + 1);
  }
  return best;
}
// O(n) time, O(k) space.
```
Delete at zero, or `count.size` counts departed types forever and the constraint never trips.
Parameterizing `k` instead of hardcoding 2 answers the generalization follow-up before it's
asked.

**27. Max Consecutive Ones III (LC 1004)** — Template B, state is one counter.

```js
function longestOnes(nums, k) {
  let l = 0, zeros = 0, best = 0;
  for (let r = 0; r < nums.length; r++) {
    if (nums[r] === 0) zeros++;
    while (zeros > k) { if (nums[l] === 0) zeros--; l++; }
    best = Math.max(best, r - l + 1);
  }
  return best;
}
// O(n) time, O(1) space.
```
`l` and `zeros` never reset, so the `while` runs at most n times overall. This, 424 and 904
are the same skeleton with three different notions of "invalid" — if that isn't obvious yet,
write all three back to back.

**28. Minimum Window Substring (LC 76)** — Template C with `need` / `window` / `formed`.

```js
function minWindow(s, t) {
  if (t.length > s.length) return '';
  const need = new Map();
  for (const c of t) need.set(c, (need.get(c) || 0) + 1);
  const win = new Map();
  let required = need.size, formed = 0;
  let l = 0, bestLen = Infinity, bestStart = 0;

  for (let r = 0; r < s.length; r++) {
    const c = s[r];
    if (need.has(c)) {
      win.set(c, (win.get(c) || 0) + 1);
      if (win.get(c) === need.get(c)) formed++;
    }
    while (formed === required) {
      if (r - l + 1 < bestLen) { bestLen = r - l + 1; bestStart = l; }
      const out = s[l];
      if (need.has(out)) {
        win.set(out, win.get(out) - 1);
        if (win.get(out) < need.get(out)) formed--;
      }
      l++;
    }
  }
  return bestLen === Infinity ? '' : s.slice(bestStart, bestStart + bestLen);
}
// O(|s| + |t|) time and space.
```
`===` when incrementing `formed`, `<` when decrementing, and **one** `slice` at the very end.
Test `s="a", t="aa"` → `""` (duplicates in `t` matter) and `s="ab", t="b"` → `"b"`.

**29. Subarray Product Less Than K (LC 713)** — counting window.

```js
function numSubarrayProductLessThanK(nums, k) {
  if (k <= 1) return 0;
  let l = 0, prod = 1, count = 0;
  for (let r = 0; r < nums.length; r++) {
    prod *= nums[r];
    while (prod >= k) { prod /= nums[l]; l++; }
    count += r - l + 1;
  }
  return count;
}
// O(n) time, O(1) space.
```
`count += r - l + 1` counts every subarray ending at `r`. The `k <= 1` guard stops `l` running
past `r`. Say the precondition: all values ≥ 1, otherwise a zero or a fraction breaks the
monotonicity that lets you shrink.

**30. Number of Subarrays with Bounded Maximum (LC 795)** — the subtraction trick again,
without a map.

```js
function numSubarrayBoundedMax(nums, left, right) {
  const atMost = (bound) => {
    let count = 0, run = 0;
    for (const x of nums) {
      run = x <= bound ? run + 1 : 0;   // subarrays ending here with every element ≤ bound
      count += run;
    }
    return count;
  };
  return atMost(right) - atMost(left - 1);
}
// O(n) time, O(1) space.
```
"Max in `[left, right]`" = "all elements ≤ right" **minus** "all elements ≤ left−1". Identical
shape to `atMost(k) − atMost(k−1)` in LC 992 — one idea, two problems. The `run = 0` reset on
a too-large element is what keeps it O(1) space instead of needing a map.

**31. Count Binary Substrings (LC 696)** — ⚠️ *not* a window. Run-length grouping.

```js
function countBinarySubstrings(s) {
  let prev = 0, cur = 1, count = 0;
  for (let i = 1; i < s.length; i++) {
    if (s[i] === s[i - 1]) cur++;
    else { count += Math.min(prev, cur); prev = cur; cur = 1; }
  }
  return count + Math.min(prev, cur);     // don't forget the final pair of runs
}
// O(n) time, O(1) space.
```
**Why no window works:** valid substrings are constrained by *run structure*, and validity
isn't monotone as you extend right — it flips on and off at every run boundary, failing
condition 2 from Part 7.2. Two adjacent runs of lengths `a` and `b` contribute exactly
`min(a, b)` valid substrings. This problem is in the set specifically to train **pattern
rejection**: "contiguous substring, count them" is not automatically a window. And don't
forget the final `Math.min` after the loop — the last run pair is never flushed inside it.

**32. First Missing Positive (LC 41)** — cyclic sort.

```js
function firstMissingPositive(nums) {
  const n = nums.length;
  for (let i = 0; i < n; i++) {
    while (nums[i] > 0 && nums[i] <= n && nums[nums[i] - 1] !== nums[i]) {
      const j = nums[i] - 1;
      [nums[i], nums[j]] = [nums[j], nums[i]];
    }
  }
  for (let i = 0; i < n; i++) if (nums[i] !== i + 1) return i + 1;
  return n + 1;
}
// O(n) time, O(1) space.
```
Two defences: the nested `while` is O(n) total because each swap permanently places a value;
and the loop condition compares **values** (`nums[nums[i]-1] !== nums[i]`) so duplicates
terminate — the index-based version loops forever on `[1,1]`. Also worth saying: the answer is
always in `1..n+1`, which is why values outside that range can be ignored. Test `[1,2,3]` → 4,
`[7,8,9]` → 1, `[3,4,-1,1]` → 2.

### If you missed any

Mark it in `log.md` and re-solve after 3 days, then after 10. The ones worth extra attention,
because they carry the most weight downstream:

| Problem | Why it matters later |
|---|---|
| 8 (3Sum) | duplicate-skipping discipline reappears in every combinatorial problem in Phase 4 |
| 10 / 28 (min window) | Template C — if this is shaky, every "shortest valid" problem is shaky |
| 11 (Subarray Sum = K) | prefix + hash map is the standard escape when a window can't handle negatives |
| 31 (Count Binary Substrings) | pattern **rejection** — knowing when *not* to reach for a window |
| 32 (First Missing Positive) | the amortized argument for a nested `while` — the same argument as sliding window |

---
## Part 19 — One-page cheat sheet

```
════ WHICH TRICK? ════════════════════════════════════════════════════
  sorted + pair/triplet sum        two pointers from the ends
  palindrome / reverse in place    two pointers from the ends
  in place, return new length      write pointer
  contiguous + longest/shortest    sliding window (variable)
  window of size k                 sliding window (fixed)
  at most k distinct / zeros       variable window, shrink while broken
  exactly k                        atMost(k) − atMost(k−1)
  count subarrays where…           window, count += r − l + 1
  range sums / sum equals k        prefix sum (+ hash map to COUNT)
  divisible by k                   prefix sum mod k → ((x%k)+k)%k
  equal #0s and #1s                map 0→−1, prefix revisits a value
  product except self              prefix × suffix
  max sum / profit / product       Kadane (running value)
  values in 1..n, O(1) space       index-as-hash / cyclic sort
  merge two sorted, in place       fill from the BACK
  0s/1s/2s in one pass             Dutch national flag (3 pointers)
  unsorted + need INDEXES          hash map (Phase 2), NOT sorting

════ THE FOUR SKELETONS ══════════════════════════════════════════════
  WRITE POINTER                    TWO POINTERS (opposite ends)
  let w = 0;                       let l = 0, r = n - 1;
  for (let r = 0; r < n; r++)      while (l < r) {
    if (keep(a[r])) a[w++] = a[r];   if (cond) l++; else r--;
  return w;                        }
  O(n)/O(1)  invariant: w <= r     O(n)/O(1)  needs SORTED

  WINDOW: LONGEST                  WINDOW: SHORTEST
  let l = 0, best = 0;             let l = 0, best = Infinity;
  for (let r = 0; r < n; r++) {    for (let r = 0; r < n; r++) {
    add(a[r]);                       add(a[r]);
    while (INVALID){rm(a[l]);l++;}   while (VALID) {
    best = max(best, r - l + 1);       best = min(best, r-l+1);
  }                                    rm(a[l]); l++;
                                     }
                                   }
                                   return best===Infinity ? 0 : best;

  LONGEST  → shrink while INVALID · record AFTER  · init 0
  SHORTEST → shrink while VALID   · record INSIDE · init Infinity

════ FIXED WINDOW ════════════════════════════════════════════════════
  build first k, then:  sum += a[i] - a[i-k]      ← THE SLIDE
  compact form:
    sum += a[i];
    if (i >= k)     sum -= a[i - k];        // the LEAVER is a[i-k]
    if (i >= k - 1) best = max(best, sum);  // COMPLETE windows only
  window start indexes:  for (i = 0; i + k <= n; i++)

════ PREFIX SUM ══════════════════════════════════════════════════════
  pre = new Array(n+1).fill(0);  pre[i+1] = pre[i] + a[i];
  sum(i..j) = pre[j+1] - pre[i]        ← ALWAYS the n+1 / pre[0]=0 form

  COUNT subarrays == k:  seen = new Map([[0,1]]);
                         sum += x;
                         count += seen.get(sum-k) || 0;   ← query BEFORE insert
                         seen.set(sum, (seen.get(sum)||0)+1);
  LONGEST subarray == k: store FIRST index → if (!map.has(sum)) map.set(sum,i)
  SHORTEST             : always overwrite
  2D: pre[i+1][j+1] = m[i][j] + pre[i][j+1] + pre[i+1][j] - pre[i][j]
      rect = pre[r2+1][c2+1] - pre[r1][c2+1] - pre[r2+1][c1] + pre[r1][c1]
  range UPDATES: diff[l] += v; diff[r+1] -= v;  then prefix-sum it

════ KADANE ══════════════════════════════════════════════════════════
  cur = best = a[0];              // NOT 0 — all-negative arrays break that
  cur  = max(a[i], cur + a[i]);   // best ENDING HERE
  best = max(best, cur);          // best ANYWHERE
  stock profit: best = 0 IS right (doing nothing is allowed) — read the problem
  max PRODUCT: track max AND min; compute both candidates BEFORE assigning

════ WHY IT'S O(n) — SAY THIS UNPROMPTED ═════════════════════════════
  r advances n times; l only moves forward, ≤ n times TOTAL
  → ≤ 2n pointer moves, not n per step
  inner counter RESETS   → multiply → O(n²)
  inner counter PERSISTS → add      → O(n)

════ WHEN THE WINDOW BREAKS ══════════════════════════════════════════
  negatives + "sum ≥ target"   → prefix sum + hash map / monotonic deque
  subsequence (skipping ok)    → DP, Phase 10
  max of every window          → monotonic deque, Phase 6
  run-structure rules (LC 696) → run-length grouping
  precondition: validity must be MONOTONE as the window grows

════ COMPLEXITY ══════════════════════════════════════════════════════
  write pointer / two pointers / window        O(n)        O(1)
  sort + two pointers                          O(n log n)
  3Sum  O(n²)      kSum  O(n^(k−1))                        O(1) aux
  window + Map                                 O(n)        O(k)
  window + 26-array                            O(n)        O(1)
  prefix sum build / query                     O(n) / O(1) O(n)
  prefix + hash map                            O(n)        O(n)
  all subarrays (brute)  O(n²), O(n³) if re-summing each

════ JS TRAPS ════════════════════════════════════════════════════════
  Math.max(...arr)     RangeError on big arrays → loop or reduce
                       (fine on a FIXED 26-element array)
  sort()               mutates + sorts as TEXT → (a,b)=>a-b, and SAY it mutates
  %                    remainder, not modulo → ((x%k)+k)%k
  slice in a loop      O(n) → O(n²).  Pass INDEXES; slice once at the end
  arr[i] out of range  undefined → NaN silently.  NaN answer = out-of-bounds
  new Array(n)         holes → use .fill(0) or Array.from
  Array(n).fill([])    ONE shared array → Array.from({length:n},()=>[])
  Object.keys().length O(n) in a loop → map.size is O(1)
  for…in on arrays     string keys + inherited props → never
  arr.length in a loop that mutates length → cache it first
  no int overflow      exact to 2^53; (lo+hi)>>1 breaks past 2^31
  strings immutable    no O(1)-space reversal; [...s] not s.split('') for emoji
  charCodeAt offset    97 = 'a' (LC 438/567) · 65 = 'A' (LC 424) — READ THE CASE
  delete arr[i]        creates a hole → never

════ SIX CHECKS BEFORE "DONE" ════════════════════════════════════════
  1  window length is r − l + 1
  2  bounds: windows i+k<=n · pairs i<n−1 · backward i>=0
  3  the leaver in a fixed window is a[i−k]
  4  record AFTER the shrink (longest) / INSIDE it (shortest)
  5  init: 0 if empty answer legal · nums[0]/−Infinity if not · Infinity to minimize
  6  l<r for distinct pairs · l<=r when the middle must be processed
     strict > keeps the FIRST tie, >= keeps the LAST

════ ALWAYS DRY-RUN THESE FIVE ═══════════════════════════════════════
  []   [5]   [2,2]/"aa"   all negatives   smallest case where answer ≠ whole array

════ INTERVIEW LINES ═════════════════════════════════════════════════
  open:   "brute force checks every pair, O(n²). It's sorted, so comparing the
           ends tells me which end can't be in the answer → O(n), O(1) space."
  window: "contiguous, and validity is monotone, so expand right / shrink left.
           Nested loop but O(n) — left never resets, ≤ n moves total."
  limits: "this needs all-positive values; with negatives I'd switch to prefix
           sums plus a hash map."
  close:  "O(n) time, O(1) auxiliary. Note it mutates the input — I'd copy first
           if the caller needs it preserved."
  stuck:  narrate the search over the signal table. Never go silent.
```

---

## Part 20 — You're done with Phase 1 when…

- [ ] You can write all four skeletons from memory, correctly, without re-deriving the
      boundary conditions.
- [ ] You can decide LONGEST vs SHORTEST within 15 seconds of reading a window problem, and
      put the recording line in the right place first try.
- [ ] You give the O(n) amortized argument for sliding window **unprompted** ("left never
      resets"), and the exchange argument for Container With Most Water.
- [ ] You can say when a window *doesn't* apply — negatives, subsequences, max-per-window,
      run-structure — and name the replacement.
- [ ] You can solve 3Sum with all three duplicate skips, cold.

When that's true, come back and say **"Start Phase 2"** — hashing: `Map`/`Set` mechanics,
frequency counters, grouping by computed key, and collapsing O(n²) brute force with O(1)
lookups. It's the shortest phase in the roadmap and the highest hit rate per hour: roughly a
third of easy/medium problems reduce to "put it in a Map."
