# Phase 1 — Arrays, Two Pointers, Sliding Window & Prefix Sums (JavaScript)

Complete reference. Everything you need for Days 3–9.

This is the highest-yield phase in the roadmap after hashing. Roughly half of the array and string problems you will be handed in a frontend interview are solved by one of four skeletons in this document: **write pointer**, **opposite-end pointers**, **sliding window**, **prefix sum**. Memorize the four skeletons, learn the signal that selects one, and most "medium" array problems stop being puzzles.

**Contents**

1. What this phase actually buys you
2. Signal → pattern decision table
3. Arrays in JavaScript — what you are actually manipulating
4. Traversal fundamentals and loop hygiene
5. In-place modification — the write-pointer pattern
6. Two pointers — opposite ends
7. Sort + two pointers — the k-sum family
8. Sliding window — the model and the two templates
9. Sliding window with counters
10. Prefix sums
11. The running-value family (Kadane and friends)
12. Index-as-hash and cyclic sort — O(1)-space array tricks
13. JS/V8 gotchas that catch candidates in this phase
14. Complexity reference for every pattern here
15. Frontend relevance — the honest version
16. Interview scripts, per pattern
17. Off-by-one discipline and the dry-run protocol
18. The 7-day plan with problem assignments
19. Practice — 32 problems
20. Worked answers
21. One-page cheat sheet

---

## 1. What this phase actually buys you

Phase 0 taught you to *measure* code. This phase teaches you to *reach* a target complexity on purpose.

Almost every problem here has the same shape:

> The obvious solution examines every pair or every subarray — O(n²) or O(n³). Find the structure that lets you avoid re-examining what you already looked at, and get to O(n) or O(n log n).

Only a few kinds of structure make that possible, and each maps to one pattern:

| The structure you exploit | Pattern it unlocks |
|---|---|
| The array is **sorted** — comparing the two ends tells you which end to discard | Two pointers, opposite ends |
| The output is a **prefix of the same array** — a write index can trail a read index | Write pointer |
| The answer is a **contiguous run**, and extending it changes validity in one direction only | Sliding window |
| You need **sums over ranges**, and a range sum is the difference of two running totals | Prefix sum |
| Values are **bounded by the array's own indices** (1..n) | Index-as-hash / cyclic sort |

When you stall in an interview, walk that table out loud. It is not filler — it is the actual search you are performing, and interviewers score it.

**Deliverable for this phase:** given an unseen array/string problem, name the pattern and state the target complexity within 60 seconds, then write the skeleton from memory without re-deriving the boundary conditions.

---

## 2. Signal → pattern decision table

Read this until it's automatic. It's the single most useful page in the document.

| What the problem says | What it almost always means |
|---|---|
| "sorted array" + "find a pair / triplet summing to X" | Two pointers from the ends (sort first if unsorted) |
| "palindrome", "reverse in place", "swap the ends" | Two pointers from the ends |
| "remove / move elements in place", "return the new length", "O(1) extra space" | Write pointer (slow/fast, same direction) |
| "contiguous subarray" or "substring" + "longest / shortest / max / min" | Sliding window, variable size |
| "subarray of size k", "window of k", "average of every k" | Sliding window, fixed size |
| "at most k distinct / at most k replacements / at most k zeros" | Variable window; shrink while the k-constraint is violated |
| "exactly k distinct" | `atMost(k) − atMost(k−1)` |
| "count subarrays where …" and the predicate is monotone | Window, adding `r − l + 1` per step |
| "sum of a range", "many range queries", "subarray sums to k" | Prefix sum (plus a hash map if you must *count* subarrays) |
| "subarray sum divisible by k" | Prefix sum modulo k, plus a hash map |
| "equal number of 0s and 1s" | Map 0 → −1, then it's prefix-sum-equals-zero |
| "product of all except self", "without division" | Prefix and suffix products |
| "maximum sum subarray", "best profit", "max product subarray" | Running-value DP (Kadane family) |
| "array holds numbers in the range 1..n" + "O(1) space" | Index-as-hash: negation marking or cyclic sort |
| "merge two sorted arrays in place" | Two pointers filling **from the back** |
| "three distinct values, sort in place, one pass" | Dutch national flag (three pointers) |
| Unsorted array, need pairs summing to a target, order irrelevant | Hash map — that's Phase 2, not two pointers |

**The distinction that matters most:** opposite-end two pointers converge and require *order* (sortedness). Sliding window moves both pointers in the *same* direction and requires *contiguity* plus a monotone validity condition. Candidates blur these together and then cannot explain why their loop terminates.

---

## 3. Arrays in JavaScript — what you are actually manipulating

A JS array is not a contiguous block of `int`s. It is an object whose keys happen to be integer-like strings, which V8 optimizes into something array-shaped *when it can*. For interviews this matters in three places: element kinds, holes, and 2D construction.

### Element kinds, and why they are one-way doors

V8 tracks an "elements kind" per array and picks a fast path from it. Simplified ladder:

```
PACKED_SMI_ELEMENTS      only small integers, no gaps      ← fastest
PACKED_DOUBLE_ELEMENTS   numbers including floats
PACKED_ELEMENTS          any values (objects, strings, mixed)
HOLEY_SMI_ELEMENTS       small integers, has gaps
HOLEY_DOUBLE_ELEMENTS
HOLEY_ELEMENTS           any values, has gaps               ← slowest
```

Transitions only go **down** the ladder. Pushing a string into a SMI array degrades it permanently, even if you remove the string afterward.

```js
const a = [1, 2, 3];   // PACKED_SMI
a.push(4.5);           // → PACKED_DOUBLE
a.push('x');           // → PACKED_ELEMENTS   (no way back)
```

None of this changes Big-O. It changes the constant factor, sometimes several-fold. You will not be quizzed on it, but "I'll keep this array numeric so V8 stays on the SMI fast path" reads as real depth in a performance discussion — *provided* you also say it doesn't change the complexity.

### Holes are the part that actually bites

A hole is a missing index, not an index holding `undefined`. Once an array is holey, every read must check the prototype chain in case someone defined `Array.prototype[5]`.

```js
const a = new Array(3);          // [ <3 empty items> ] → HOLEY, length 3, zero elements
a[0];                            // undefined — via a slow path
0 in a;                          // false    ← the tell

const b = new Array(3).fill(0);              // [0,0,0] → PACKED_SMI ✅
const c = Array.from({ length: 3 }, () => 0); // also packed ✅

const d = [1, 2, 3];
delete d[1];                     // [1, <1 empty item>, 3] → HOLEY. Never do this.
d.length;                        // still 3
```

And the iteration surprises:

```js
const holey = [1, , 3];
holey.map(x => x * 2);           // [2, <1 empty item>, 6]  — map SKIPS holes
holey.forEach(() => {});         // callback runs twice, not three times
holey.includes(undefined);       // true   (includes treats holes as undefined)
holey.indexOf(undefined);        // -1     (indexOf skips holes)
```

**Rule for this phase:** allocate result arrays with `new Array(n).fill(0)` or `Array.from`. Never bare `new Array(n)`, never `delete` on an array.

### 2D arrays — the shared-reference bug

```js
const grid = Array(3).fill([]);   // ❌ all three rows are the SAME array
grid[0].push(1);
grid;                             // [[1], [1], [1]]

const ok = Array.from({ length: 3 }, () => []);                          // ✅
const dp = Array.from({ length: rows }, () => new Array(cols).fill(0));  // ✅
```

`fill` with an object fills with one reference; `Array.from` with a factory calls it per index. This bug shows up in DP tables and grids (Phases 9 and 10) and in real React code. Get it right by reflex.

### Typed arrays — when they're worth mentioning

`Int32Array`, `Float64Array`, `Uint8Array` are genuinely contiguous, fixed length, zero-initialized, and cannot have holes.

```js
const counts = new Uint8Array(26);   // zero-filled, packed, hole-proof
```

Same Big-O, better constants, lower memory. Worth one sentence when a problem says "the array can hold 10⁷ integers." Not worth reaching for in a 20-minute interview where clarity beats bytes.

### Cost recap — the subset of Phase 0 you need here

| Operation | Cost | Note for this phase |
|---|---|---|
| `arr[i]`, `arr.length` | O(1) | index math is always cheaper than slicing |
| `push` / `pop` | O(1) amortized | fine inside loops |
| `shift` / `unshift` / `splice` | **O(n)** | never inside a loop; use a write pointer |
| `slice`, `concat`, `[...arr]` | **O(n)** | never inside a window loop — this is how O(n) becomes O(n²) |
| `includes` / `indexOf` / `find` | **O(n)** | inside a loop → O(n²); use a `Set`/`Map` |
| `sort((a,b) => a-b)` | O(n log n) | mutates the input — say so out loud |
| `map` / `filter` / `reduce` | O(n) time, O(n) space | allocates; in-place problems forbid them |

The patterns in this phase exist precisely to avoid the O(n) rows of that table. If your "O(n) sliding window" calls `slice` to materialize the current window, it is O(n²). That is the most common self-inflicted wound in this phase.

---

## 4. Traversal fundamentals and loop hygiene

Boring section. Most bugs live here.

### Which loop to use

```js
for (let i = 0; i < arr.length; i++) { }   // need the index, or need to jump → this
for (const x of arr) { }                    // values only, no index math → clean
arr.forEach((x, i) => { });                 // cannot break → avoid in algorithm code
for (const i in arr) { }                    // ❌ never on arrays: keys are STRINGS, includes inherited
```

The `for...in` trap, concretely:

```js
const a = [10, 20, 30];
for (const i in a) console.log(i + 1);   // "01", "11", "21" — string concatenation
```

Every pattern in this phase wants explicit index loops. Pointers *are* indices; hiding them behind an iterator makes the pattern unwritable.

### Empty and single-element inputs

```js
let r = arr.length - 1;        // -1 when empty
while (l < r) { }              // 0 < -1 is false → loop skipped. Often correct by accident.
arr[arr.length - 1];           // undefined on empty — silent wrong answer, not a crash
Math.max(...[]);               // -Infinity
[].reduce((a, b) => a + b);    // ❌ TypeError: Reduce of empty array with no initial value
```

`while (l < r)` loops are usually empty-safe for free. Anything that reads `arr[0]` or `arr[n-1]` before the loop is not. Say the guard out loud — "empty returns 0, single element returns itself" — then write it.

### The swap

```js
[a[i], a[j]] = [a[j], a[i]];              // idiomatic, O(1) — write this
const t = a[i]; a[i] = a[j]; a[j] = t;    // marginally faster in a hot loop
```

Use the destructuring form. If asked about the array allocation, say V8's escape analysis normally removes it and it's O(1) either way.

### Bounds patterns to recognize instantly

```js
for (let i = 0; i < n; i++)          // every element
for (let i = 1; i < n; i++)          // every element compared to i-1
for (let i = 0; i < n - 1; i++)      // every adjacent PAIR (i, i+1)
for (let i = n - 1; i >= 0; i--)     // right to left — suffix values, or writing without overwriting
for (let i = 0; i + k <= n; i++)     // every window of size k, by start index
```

Windows use `i + k <= n`, not `i + k < n`. The last valid window starts at `n - k`, and `(n-k) + k <= n` holds. Getting this wrong silently drops the final window — a very common bug, and the interviewer's test case usually catches it.

### Backward comparison beats forward

Prefer `arr[i-1]` with the loop starting at 1 over `arr[i+1]` with the loop ending at `n-1`. Backward reads can never go out of bounds when the start index is right, and the previous value is the one you've already validated.

---
## 5. In-place modification — the write-pointer pattern

The first of the four skeletons. Also called slow/fast pointers or the read/write pointer.

**When:** the problem says "in place", "return the new length", "O(1) extra space", or "the order of the remaining elements may/must be preserved."

**The idea:** one pointer reads every element exactly once. A second pointer marks where the next *kept* element belongs. Because the write pointer never overtakes the read pointer, you can overwrite in place with no buffer.

### Template — memorize this

```js
function keepIf(arr, shouldKeep) {
  let w = 0;                       // write index = count of kept elements so far
  for (let r = 0; r < arr.length; r++) {
    if (shouldKeep(arr[r], w, arr)) {
      arr[w] = arr[r];
      w++;
    }
  }
  return w;                        // arr[0..w-1] is the answer
}
// Time O(n) — each element read once, written at most once.
// Space O(1) — two indices.
```

Two invariants do all the work, and saying them out loud is most of the credit:

1. `arr[0 .. w-1]` is always the final answer for everything processed so far.
2. `w <= r` always — so writing to `w` can never clobber an element you still need to read.

### Remove Element (LC 27)

```js
function removeElement(nums, val) {
  let w = 0;
  for (let r = 0; r < nums.length; r++) {
    if (nums[r] !== val) nums[w++] = nums[r];
  }
  return w;
}
// O(n) time, O(1) space.
```

The order-doesn't-matter variant swaps the last element in instead, which does fewer writes when matches are rare:

```js
function removeElementSwap(nums, val) {
  let n = nums.length, i = 0;
  while (i < n) {
    if (nums[i] === val) nums[i] = nums[--n];   // pull from the end; don't advance i
    else i++;
  }
  return n;
}
// O(n) time, O(1) space. Note: do NOT i++ after pulling — the pulled value is unchecked.
```

That "don't advance after replacing" detail is the same trap as the `high` pointer in Dutch national flag below. Whenever you move an *unexamined* element into the current slot, you must re-examine that slot.

### Remove Duplicates from Sorted Array (LC 26)

The predicate becomes "different from the last thing I kept" — that's why the template passes `w` to `shouldKeep`.

```js
function removeDuplicates(nums) {
  if (nums.length === 0) return 0;
  let w = 1;                                   // first element is always kept
  for (let r = 1; r < nums.length; r++) {
    if (nums[r] !== nums[w - 1]) nums[w++] = nums[r];
  }
  return w;
}
// O(n) time, O(1) space.
```

Compare against `nums[w-1]` (the last *kept* value), not `nums[r-1]` (the last *read* value). On sorted input both happen to work; the moment the rule becomes "at most two copies" only the kept-value form generalizes.

### Remove Duplicates from Sorted Array II — at most k copies (LC 80)

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

This is the whole family in three lines. `nums[w-k]` is the value `k` positions back in the *output*; if the incoming value differs from it, fewer than `k` copies have been written. Generalizing "at most 2" to "at most k" with one parameter is a strong signal in an interview — it shows you understood the invariant rather than memorizing the case.

### Move Zeroes (LC 283)

Keep the non-zeros in order, then fill the tail:

```js
function moveZeroes(nums) {
  let w = 0;
  for (let r = 0; r < nums.length; r++) {
    if (nums[r] !== 0) nums[w++] = nums[r];
  }
  while (w < nums.length) nums[w++] = 0;
}
// O(n) time, O(1) space. Writes: up to n non-zeros + trailing zeros.
```

Or one pass with a swap, which is the version to mention when the interviewer asks about write count:

```js
function moveZeroesSwap(nums) {
  let w = 0;
  for (let r = 0; r < nums.length; r++) {
    if (nums[r] !== 0) {
      if (w !== r) [nums[w], nums[r]] = [nums[r], nums[w]];
      w++;
    }
  }
}
// O(n) time, O(1) space. Minimum number of writes — relevant if writes are expensive (flash memory, DOM).
```

The `if (w !== r)` guard avoids self-swaps. In a DOM context (reordering nodes) that guard is the difference between reflowing every node and reflowing only the moved ones — a legitimate frontend point to raise.

### What NOT to do

```js
// ❌ O(n²): splice shifts every later element on each removal
for (let i = 0; i < nums.length; i++) {
  if (nums[i] === val) { nums.splice(i, 1); i--; }
}

// ❌ O(n) space, and fails "in place, return the new length"
return nums.filter(x => x !== val);
```

Both are what an untrained candidate writes. Naming *why* they're wrong — `splice` is O(n) per call, `filter` allocates — is the point of Phase 0.

---

## 6. Two pointers — opposite ends

Second skeleton. Two indices start at the two ends and move toward each other. Each iteration moves at least one pointer, and they never move backward, so the loop runs at most n times → **O(n)**.

### Template

```js
let l = 0, r = arr.length - 1;
while (l < r) {
  // decide, from arr[l] and arr[r], which side can be safely discarded
  if (someCondition) l++;
  else r--;
}
// O(n) time, O(1) space.
```

**`l < r` vs `l <= r`:** use `l < r` when you need two *distinct* positions (pairs, palindrome, area). Use `l <= r` only when the single middle element must itself be processed (that's binary search, Phase 3). Picking the wrong one gives you an off-by-one on odd-length inputs — the classic sighting is a palindrome check that compares the middle character to itself, which is harmless, versus a pair-sum that returns `[i, i]`, which is wrong.

**The correctness question you must be able to answer:** *why is discarding a whole side safe?* Every problem below has a different one-sentence answer, and that sentence is what separates "I've seen this" from "I understand this."

### Reverse in place

```js
function reverse(arr) {
  let l = 0, r = arr.length - 1;
  while (l < r) {
    [arr[l], arr[r]] = [arr[r], arr[l]];
    l++; r--;
  }
  return arr;
}
// O(n) time, O(1) space.  n/2 swaps → still O(n).
```

Strings are immutable in JS, so string reversal is `[...str].reverse().join('')` — O(n) time and O(n) space, and there is no O(1)-space option. Use `[...str]` rather than `str.split('')` so surrogate pairs survive (Section 13).

### Valid Palindrome (LC 125)

```js
function isPalindrome(s) {
  const isAlnum = c => /[a-z0-9]/i.test(c);
  let l = 0, r = s.length - 1;
  while (l < r) {
    while (l < r && !isAlnum(s[l])) l++;
    while (l < r && !isAlnum(s[r])) r--;
    if (s[l].toLowerCase() !== s[r].toLowerCase()) return false;
    l++; r--;
  }
  return true;
}
// O(n) time, O(1) space.
```

Three things interviewers look for here:

- The inner `while`s carry `l < r` too. Without it, an all-punctuation string runs the pointer off the end.
- The O(1) space version skips non-alphanumerics in place. The easy version — `s.replace(/[^a-z0-9]/gi,'').toLowerCase()` then compare to its reverse — is O(n) space. Both are O(n) time; mention the trade and pick based on what they asked for.
- The nested `while` here does **not** make it O(n²): `l` and `r` only ever move toward each other, so total movement ≤ n. This is the resets-vs-persists test from Phase 0, and it's worth saying explicitly.

### Two Sum II — sorted input (LC 167)

```js
function twoSum(nums, target) {
  let l = 0, r = nums.length - 1;
  while (l < r) {
    const sum = nums[l] + nums[r];
    if (sum === target) return [l + 1, r + 1];   // LC 167 wants 1-indexed
    if (sum < target) l++;                        // need bigger → only l can grow it
    else r--;                                     // need smaller → only r can shrink it
  }
  return [];
}
// O(n) time, O(1) space.
```

**Why discarding is safe:** if `nums[l] + nums[r] < target`, then `nums[l]` paired with *anything* at or left of `r` is even smaller — `nums[l]` cannot be part of any solution within `[l, r]`, so `l` is done forever. Symmetrically for `r`. Being able to state that is the whole problem.

Contrast with unsorted Two Sum (LC 1), which is a hash map problem: O(n) time, O(n) space, no sorting. If you sort an unsorted input to use two pointers you pay O(n log n) *and* you destroy the original indices — fatal when the answer must be indices. Say this trade-off out loud; it's a favorite follow-up.

### Container With Most Water (LC 11)

```js
function maxArea(height) {
  let l = 0, r = height.length - 1, best = 0;
  while (l < r) {
    best = Math.max(best, (r - l) * Math.min(height[l], height[r]));
    if (height[l] < height[r]) l++;
    else r--;
  }
  return best;
}
// O(n) time, O(1) space.
```

**Why moving the shorter side is safe** — the exchange argument, and you should be able to give it in two sentences:

The area is `width × min(h[l], h[r])`, so it's capped by the shorter wall. Any other container that still uses the shorter wall has a strictly smaller width and a height still capped by that same shorter wall, so it can never beat the area you just recorded — therefore the shorter wall can be discarded.

Move the *shorter* side. Candidates who move the taller side, or move both, produce something that passes some tests and fails others, and cannot say why.

### Trapping Rain Water (LC 42)

Worth learning here even though it's tagged hard — it's the best demonstration of converting prefix arrays into O(1) space.

Start from the honest, easy version:

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
// O(n) time, O(n) space.  Water above bar i = min(tallest to the left, tallest to the right) − h[i].
```

Then compress it. You never need both maxima in full — only the smaller one, and two pointers can track that:

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

**Why it's correct:** when `h[l] < h[r]`, some bar at or before `r` is at least `h[r] > h[l]`, so the *right* max for index `l` is guaranteed ≥ `h[r]`, which means `min(leftMax, rightMax) = leftMax`. You can commit to the left side's water without ever computing the right max exactly. Showing the O(n)-space version first and then compressing it is a strong interview move — it demonstrates the derivation rather than a memorized trick.

### Squares of a Sorted Array (LC 977)

Sorted but containing negatives: the largest square is at one of the two ends. Fill the output back to front.

```js
function sortedSquares(nums) {
  const n = nums.length, res = new Array(n);
  let l = 0, r = n - 1;
  for (let w = n - 1; w >= 0; w--) {
    const a = nums[l] * nums[l], b = nums[r] * nums[r];
    if (a > b) { res[w] = a; l++; }
    else { res[w] = b; r--; }
  }
  return res;
}
// O(n) time, O(n) space for the output (O(1) auxiliary).
// Sorting the squares would be O(n log n) — the two-pointer version is the point of the problem.
```

### Merge Sorted Array (LC 88) — fill from the back

`nums1` has `m` values plus `n` slots of room. Merging forward would overwrite unread values; merging backward writes into slots that are already free.

```js
function merge(nums1, m, nums2, n) {
  let i = m - 1, j = n - 1, w = m + n - 1;
  while (j >= 0) {
    nums1[w--] = (i >= 0 && nums1[i] > nums2[j]) ? nums1[i--] : nums2[j--];
  }
}
// O(m + n) time, O(1) space.
```

Loop on `j >= 0` only: once `nums2` is exhausted, whatever remains in `nums1` is already in place. This "write backward to avoid clobbering" instinct is the generalizable lesson — it comes back in string manipulation and in-place array shifting problems.

### Dutch national flag / Sort Colors (LC 75) — three pointers

Partition into `<`, `==`, `>` in one pass.

```js
function sortColors(nums) {
  let low = 0, mid = 0, high = nums.length - 1;
  while (mid <= high) {
    if (nums[mid] === 0) {
      [nums[low], nums[mid]] = [nums[mid], nums[low]];
      low++; mid++;
    } else if (nums[mid] === 1) {
      mid++;
    } else {
      [nums[mid], nums[high]] = [nums[high], nums[mid]];
      high--;                        // ⚠️ do NOT mid++ here
    }
  }
}
// O(n) time, O(1) space. One pass.
```

Invariants: `[0, low)` is all 0s, `[low, mid)` is all 1s, `(high, n-1]` is all 2s, `[mid, high]` is unexamined.

**The one trap:** don't advance `mid` in the `=== 2` branch. The value swapped in from `high` has never been examined. Swapping from `low` is different — that region holds only 1s, which is why `mid++` is safe there. If you can explain that asymmetry you have understood the algorithm; it is the single question interviewers ask about this one.

Note `mid <= high`, not `mid < high` — with `<` you leave the final unexamined element unsorted.

---

## 7. Sort + two pointers — the k-sum family

The bridge pattern: sorting costs O(n log n) but *creates* the order that two pointers need. Once the array is sorted, k-sum reduces to (k−2) nested loops around a two-pointer scan.

### 3Sum (LC 15)

```js
function threeSum(nums) {
  nums.sort((a, b) => a - b);            // O(n log n)
  const res = [];
  const n = nums.length;

  for (let i = 0; i < n - 2; i++) {
    if (nums[i] > 0) break;                              // sorted: no way to reach 0 with three positives
    if (i > 0 && nums[i] === nums[i - 1]) continue;      // skip duplicate FIRST elements

    let l = i + 1, r = n - 1;
    while (l < r) {
      const sum = nums[i] + nums[l] + nums[r];
      if (sum < 0) l++;
      else if (sum > 0) r--;
      else {
        res.push([nums[i], nums[l], nums[r]]);
        while (l < r && nums[l] === nums[l + 1]) l++;     // skip duplicate SECOND elements
        while (l < r && nums[r] === nums[r - 1]) r--;     // skip duplicate THIRD elements
        l++; r--;
      }
    }
  }
  return res;
}
// Time O(n²) — the sort is dominated by the n × O(n) scan.
// Space O(1) auxiliary (excluding the output and the sort's internal O(log n)–O(n)).
```

Everything above the duplicate handling is easy. **The duplicate handling is the entire problem**, and it's where interviews are lost. Three separate skips, each for a different reason:

| Skip | Guard | Why |
|---|---|---|
| First element | `if (i > 0 && nums[i] === nums[i-1]) continue;` | The same `i` value would regenerate every triplet found before |
| Second element | `while (l < r && nums[l] === nums[l+1]) l++;` | Only after recording a hit — otherwise you'd skip valid distinct triplets |
| Third element | `while (l < r && nums[r] === nums[r-1]) r--;` | Same reason, from the right |

Two rules to keep straight:

- The `i` skip compares to `i-1` (backward), not `i+1`. Comparing forward skips the *first* occurrence, which is the one you actually want to process.
- The `l`/`r` skips happen only inside the `sum === 0` branch. Hoisting them out of the branch is a common wrong "optimization" that drops valid answers.

The `nums[i] > 0` break is a genuine constant-factor win on real inputs and free to justify: with the array sorted, if the smallest of the three is positive the sum can't be 0.

Alternative dedup, worth having as a backup if you blank on the skip logic under pressure: collect triplets and key them into a `Set` as `a,b,c` strings. It's O(n²) time still, but O(n²) space in the worst case, and it looks like an admission that you can't manage the pointers. Use it only as a fallback, and say that's what you're doing.

### 3Sum Closest (LC 16)

```js
function threeSumClosest(nums, target) {
  nums.sort((a, b) => a - b);
  const n = nums.length;
  let best = nums[0] + nums[1] + nums[2];
  for (let i = 0; i < n - 2; i++) {
    let l = i + 1, r = n - 1;
    while (l < r) {
      const sum = nums[i] + nums[l] + nums[r];
      if (Math.abs(sum - target) < Math.abs(best - target)) best = sum;
      if (sum === target) return sum;      // exact: cannot do better, exit early
      if (sum < target) l++;
      else r--;
    }
  }
  return best;
}
// O(n²) time, O(1) space.
```

No dedup needed — you're returning a value, not a set of distinct triplets. Initialize `best` from actual elements, never from `0` or `Infinity`: `Infinity` breaks `Math.abs(best - target)` arithmetic, and `0` is simply a wrong answer when no sum is near zero.

### 3Sum Smaller (LC 259) — the counting trick

Count triplets with sum `< target`.

```js
function threeSumSmaller(nums, target) {
  nums.sort((a, b) => a - b);
  let count = 0;
  for (let i = 0; i < nums.length - 2; i++) {
    let l = i + 1, r = nums.length - 1;
    while (l < r) {
      if (nums[i] + nums[l] + nums[r] < target) {
        count += r - l;      // ⭐ every index in (l, r] also works, since the array is sorted
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

`count += r - l` is the reusable idea: when a sorted pair satisfies the bound, *all* pairs between them satisfy it too, so you count a whole block in O(1) instead of enumerating it. The identical trick appears in Subarray Product Less Than K (Section 9) — recognize it as one pattern, not two.

### 4Sum (LC 18) and the general k-sum

```js
function fourSum(nums, target) {
  nums.sort((a, b) => a - b);
  const n = nums.length, res = [];
  for (let i = 0; i < n - 3; i++) {
    if (i > 0 && nums[i] === nums[i - 1]) continue;
    for (let j = i + 1; j < n - 2; j++) {
      if (j > i + 1 && nums[j] === nums[j - 1]) continue;
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

Note the second guard is `j > i + 1`, not `j > 0` — the dedup compares against the previous value *within this level's range*, and level `j` starts at `i+1`.

**The general statement to have ready:** k-sum on a sorted array is O(n^(k−1)) — fix `k−2` elements with nested loops, then a linear two-pointer scan. The known lower bound for 3Sum with subquadratic algorithms is a research-level topic; the expected interview answer is O(n²), and O(n^(k−1)) for the general case. Saying that sentence answers the "can you generalize?" follow-up completely.

For very large `k` you'd switch to hashing pair sums (O(n²) space) — mention it exists, don't implement it unless asked.

---
## 8. Sliding window — the model and the two templates

Third skeleton, and the highest-frequency pattern in frontend interviews. Days 6 and 7 exist for this.

### What makes a problem a window problem

Two conditions, both required:

1. **The answer is a contiguous run** — a subarray or substring, not a subsequence. If elements can be skipped, this is not a window (it's DP, Phase 10).
2. **Validity is monotone in the window** — extending the window on the right can only push it toward invalid, and shrinking from the left can only push it toward valid. Nothing about the window can flip back and forth as you add elements.

If condition 2 fails, sliding window silently produces wrong answers rather than failing loudly. The canonical failure: "shortest subarray with sum ≥ target" where **negatives are allowed**. Adding an element can *decrease* the sum, so a too-small window isn't necessarily fixable by growing and a too-large one isn't necessarily improvable by shrinking. That problem (LC 862) needs prefix sums plus a monotonic deque, not a window. Knowing *when the pattern doesn't apply* is a senior-level signal — junior candidates apply windows to anything containing the word "subarray."

Quick test to run before you commit: *"if this window is invalid, does adding another element on the right ever make it valid again?"* If yes, it's not a window problem.

### Why it's O(n) — say this out loud, unprompted

`right` advances exactly n times. `left` only ever increases and can advance at most n times **across the whole run**, not per iteration. Total pointer movement ≤ 2n → **O(n)**, despite the nested `while`.

This is Phase 0's resets-vs-persists test. The wrong answer — "there's a nested loop so it's O(n²)" — is the single most common complexity mistake in this phase, and interviewers listen for it specifically.

### Template A — fixed-size window (size k)

```js
function fixedWindow(arr, k) {
  let sum = 0, best = -Infinity;
  for (let i = 0; i < arr.length; i++) {
    sum += arr[i];                          // add the entering element
    if (i >= k) sum -= arr[i - k];          // remove the leaving element, once the window is full
    if (i >= k - 1) best = Math.max(best, sum);   // record only for complete windows
  }
  return best;
}
// O(n) time, O(1) space.
```

Three details, all of which are common bugs:

- `if (i >= k)` removes `arr[i-k]`, the element that just fell out — not `arr[i-k+1]`.
- `if (i >= k - 1)` gates recording, because windows before that are incomplete. Recording early gives you the max of a *partial* prefix, which is often larger than any real window when values are positive — a wrong answer that looks plausible.
- Guard `k > arr.length` explicitly if the problem allows it; otherwise you return `-Infinity`.

The explicit two-phase form is easier to get right under pressure and worth preferring:

```js
function fixedWindowTwoPhase(arr, k) {
  if (k > arr.length) return 0;
  let sum = 0;
  for (let i = 0; i < k; i++) sum += arr[i];       // build the first window
  let best = sum;
  for (let i = k; i < arr.length; i++) {
    sum += arr[i] - arr[i - k];                    // slide by one: add new, drop old
    best = Math.max(best, sum);
  }
  return best;
}
// O(n) time, O(1) space.
```

`sum += arr[i] - arr[i-k]` in one line *is* the slide. If you can write that line without hesitating, fixed windows are done.

### Template B — variable window, LONGEST (maximize)

```js
function longestValid(arr) {
  let l = 0, best = 0;
  for (let r = 0; r < arr.length; r++) {
    // 1. include arr[r] in the window state
    while (/* window is INVALID */) {
      // 2. remove arr[l] from the window state
      l++;
    }
    // 3. here the window is guaranteed valid → record
    best = Math.max(best, r - l + 1);
  }
  return best;
}
// O(n) time. Space = whatever the window state costs (O(1), O(k), or O(charset)).
```

### Template C — variable window, SHORTEST (minimize)

```js
function shortestValid(arr) {
  let l = 0, best = Infinity;
  for (let r = 0; r < arr.length; r++) {
    // 1. include arr[r] in the window state
    while (/* window is VALID */) {
      best = Math.min(best, r - l + 1);   // record BEFORE shrinking away validity
      // 2. remove arr[l] from the window state
      l++;
    }
  }
  return best === Infinity ? 0 : best;
}
// O(n) time.
```

### The distinction to burn in

| | Longest | Shortest |
|---|---|---|
| Inner loop shrinks while… | the window is **invalid** | the window is **valid** |
| Record the answer… | **after** the inner loop | **inside** the inner loop |
| Initialize best to | `0` | `Infinity` |
| Convert missing answer | already `0` | `Infinity → 0` at the end |

That's it. Same skeleton, mirrored. Almost every window problem is one of these two with a different notion of "window state" and "valid." When you sit down to a window problem, first decide *longest or shortest*, then write the matching skeleton, then fill in the state. Deciding those in that order prevents the most common failure — recording the answer in the wrong place, which produces off-by-one results that pass the sample tests.

**Why `while` and not `if`:** one entering element can force multiple removals (a large number entering a sum-bounded window may require dropping several small ones). An `if` shrinks at most once per step and quietly produces wrong answers. There is exactly one problem in this phase where a single `if` is defensible (Longest Repeating Character Replacement, Section 9) and even there the `while` version is safer.

### Minimum Size Subarray Sum (LC 209) — Template C

```js
function minSubArrayLen(target, nums) {
  let l = 0, sum = 0, best = Infinity;
  for (let r = 0; r < nums.length; r++) {
    sum += nums[r];
    while (sum >= target) {                 // valid → try to shrink
      best = Math.min(best, r - l + 1);
      sum -= nums[l];
      l++;
    }
  }
  return best === Infinity ? 0 : best;
}
// O(n) time, O(1) space.
```

Requires all positive values. State that precondition out loud — it's exactly the monotonicity requirement from the top of this section, and volunteering it is how you show you know the pattern's limits.

### Fixed window, minimum: Maximum Average Subarray I (LC 643)

```js
function findMaxAverage(nums, k) {
  let sum = 0;
  for (let i = 0; i < k; i++) sum += nums[i];
  let best = sum;
  for (let i = k; i < nums.length; i++) {
    sum += nums[i] - nums[i - k];
    best = Math.max(best, sum);
  }
  return best / k;                 // divide ONCE at the end
}
// O(n) time, O(1) space.
```

Dividing once at the end rather than per window avoids accumulating floating-point error and is one fewer operation per iteration. Small, but the kind of thing that reads as care.

---

## 9. Sliding window with counters

Once the window state is richer than a running sum, you need a `Map`, a `Set`, or a fixed-size count array. This is where most real window problems live.

### Choosing the window state

| Window state | Use when | Cost |
|---|---|---|
| running number (sum/product) | numeric constraint | O(1) |
| `Set` | only presence matters ("no duplicates") | O(k) space, O(1) ops |
| `Map` | you need counts, or last-seen indices | O(k) space, O(1) ops |
| `new Array(26).fill(0)` | lowercase letters only | O(1) space (26 is a constant), fastest constants |
| `Uint8Array(128)` | full ASCII, counts ≤ 255 | O(1) space |

**Never use a plain object plus `Object.keys().length` for the distinct count inside the loop** — that's O(k) per iteration and turns your O(n) into O(n·k). `map.size` is O(1). This is Phase 0's Trap 6, and window problems are where it does the most damage.

### Longest Substring Without Repeating Characters (LC 3) — Template B with a Set

```js
function lengthOfLongestSubstring(s) {
  const seen = new Set();
  let l = 0, best = 0;
  for (let r = 0; r < s.length; r++) {
    while (seen.has(s[r])) {          // invalid: duplicate present
      seen.delete(s[l]);
      l++;
    }
    seen.add(s[r]);
    best = Math.max(best, r - l + 1);
  }
  return best;
}
// O(n) time — each character is added once and deleted at most once.
// O(min(n, charset)) space.
```

Order matters: shrink **before** adding `s[r]`, or you'll delete the character you just inserted and loop forever on `"aa"`. Dry-run `"aa"` and `"abba"` before you claim it works.

The Map-of-last-index variant jumps `left` instead of stepping it:

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
// O(n) time, O(min(n, charset)) space.
```

**The `Math.max` is the whole trap.** The map keeps indices of characters that have already left the window, so a stale index can be *behind* `l`. Without the clamp, `left` moves backward and the window grows past a duplicate. Test `"abba"`: at the final `a`, `last.get('a')` is 0 → without `Math.max`, `l` becomes 1 while it should stay at 2, and you return 3 instead of 2. The Set version has no such hazard, which is why it's the better one to write live; know the Map version because interviewers ask "can you avoid the inner loop?"

### Fixed window + counts: Find All Anagrams in a String (LC 438)

```js
function findAnagrams(s, p) {
  const res = [];
  if (p.length > s.length) return res;
  const need = new Array(26).fill(0), win = new Array(26).fill(0);
  const idx = c => c.charCodeAt(0) - 97;

  for (const c of p) need[idx(c)]++;

  let matches = 0;                                  // number of letters whose counts agree
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
// O(n) time, O(1) space (two 26-element arrays).
```

That's the O(1)-per-step version. The simpler variant compares the two 26-arrays each step: still O(26n) = O(n), and much easier to write correctly under pressure.

```js
// Simpler, same complexity class — prefer this live.
function findAnagramsSimple(s, p) {
  const res = [], k = p.length;
  if (k > s.length) return res;
  const need = new Array(26).fill(0), win = new Array(26).fill(0);
  const idx = c => c.charCodeAt(0) - 97;
  for (const c of p) need[idx(c)]++;
  for (let r = 0; r < s.length; r++) {
    win[idx(s[r])]++;
    if (r >= k) win[idx(s[r - k])]--;
    if (r >= k - 1 && need.every((v, i) => v === win[i])) res.push(r - k + 1);
  }
  return res;
}
// O(26n) = O(n) time, O(1) space.
```

Say the complexity as "O(26n), which is O(n) — the 26 is a constant because the alphabet is fixed." That sentence shows you know the difference between a constant factor and a variable, which is exactly what Rule 1 of Phase 0 is about. **Permutation in String (LC 567)** is this problem returning a boolean on the first hit.

### Minimum Window Substring (LC 76) — Template C with counts

```js
function minWindow(s, t) {
  if (t.length > s.length) return '';
  const need = new Map();
  for (const c of t) need.set(c, (need.get(c) || 0) + 1);

  let required = need.size;        // distinct chars still not satisfied
  let formed = 0;
  const win = new Map();
  let l = 0, bestLen = Infinity, bestStart = 0;

  for (let r = 0; r < s.length; r++) {
    const c = s[r];
    if (need.has(c)) {
      win.set(c, (win.get(c) || 0) + 1);
      if (win.get(c) === need.get(c)) formed++;      // === not >=, so it counts exactly once
    }

    while (formed === required) {                     // valid → shrink
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
// O(|s| + |t|) time, O(|s| + |t|) space.
```

Three points that decide whether this works:

- Track `formed` as a counter. Comparing whole maps per step would be O(charset) per step — still linear-ish here, but the counter is the intended solution and it generalizes.
- `if (win.get(c) === need.get(c)) formed++` uses `===`, not `>=`. With `>=`, extra copies of a character inflate `formed` past `required` and you emit garbage windows.
- Record `bestStart`/`bestLen`, and `slice` **once** at the end. Slicing inside the loop would make it O(n²) — the same trap flagged in Section 3.

This is the hardest problem in the phase. If you can write it from the Template C skeleton without looking, variable windows are genuinely done.

### Longest Repeating Character Replacement (LC 424) — and the honest version of the maxFreq argument

Window is valid when `windowLength − maxFreqInWindow <= k`: keep the most common letter in the window, replace all the others.

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
// Math.max over a FIXED 26-element array is O(1). This is safe here precisely
// because the array length is a constant — see the Math.max(...arr) warning in Section 13.
```

**The offset trap, and it's a nasty one.** `charCodeAt(0) - 97` maps `'a'` → 0; `- 65` maps `'A'` → 0. LC 438 and 567 are lowercase; LC 424 is uppercase. Use the wrong offset and you index `count[-32]`, which is `undefined` — then `Math.max(...count)` is `NaN`, every comparison with `NaN` is false, the window never shrinks, and you get a silently wrong answer instead of a crash. **Read the constraints for case before you pick the offset.** If the input can be mixed case, index by the raw code into `new Array(128).fill(0)`, or use a `Map` and accept O(1)-with-worse-constants.

**The subtlety you'll see everywhere else.** The widely-copied solution never decrements a cached `maxFreq`, and shrinks with `if` instead of `while`:

```js
let maxFreq = 0;
for (let r = 0; r < s.length; r++) {
  count[idx(s[r])]++;
  maxFreq = Math.max(maxFreq, count[idx(s[r])]);   // never decreases
  if (r - l + 1 - maxFreq > k) { count[idx(s[l])]--; l++; }
  best = Math.max(best, r - l + 1);
}
```

It returns the right answer, and the reason is not obvious: `maxFreq` may be stale (too large), which makes the validity check too *permissive*, so the window can be genuinely invalid at some steps. But the window size never shrinks — each step either grows it by one or holds it — and it can only grow when the check passes. A window larger than the true answer would require a genuinely larger `maxFreq` than any real window contains, which never happens. So the recorded maximum is never larger than some genuinely valid window's size.

**What to do in an interview:** write the recompute version. It's the same O(n), it needs no argument, and if the interviewer brings up the cached-maxFreq trick you can explain why it also works. Reciting a memorized trick whose correctness you can't defend is worse than a clean solution — this is exactly the case where the textbook "optimization" is a trap.

### At-most-k windows

The same skeleton with different state. These three are one problem.

```js
// Max Consecutive Ones III (LC 1004): longest window with at most k zeros
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

```js
// Longest substring with at most k distinct characters (LC 340 / Fruit Into Baskets LC 904 is k = 2)
function atMostKDistinct(s, k) {
  const count = new Map();
  let l = 0, best = 0;
  for (let r = 0; r < s.length; r++) {
    count.set(s[r], (count.get(s[r]) || 0) + 1);
    while (count.size > k) {                        // map.size is O(1) ⭐
      const out = s[l];
      const c = count.get(out) - 1;
      if (c === 0) count.delete(out); else count.set(out, c);
      l++;
    }
    best = Math.max(best, r - l + 1);
  }
  return best;
}
// O(n) time, O(k) space.
```

Delete the key when its count hits zero — otherwise `count.size` counts characters that already left the window and the constraint never trips. This single line is the most common bug in the at-most-k family.

### Counting windows: add `r − l + 1`

When the question is "how many subarrays satisfy X," the count of valid subarrays *ending at r* is the current window length.

```js
// Subarray Product Less Than K (LC 713) — requires all nums[i] >= 1
function numSubarrayProductLessThanK(nums, k) {
  if (k <= 1) return 0;
  let l = 0, prod = 1, count = 0;
  for (let r = 0; r < nums.length; r++) {
    prod *= nums[r];
    while (prod >= k) { prod /= nums[l]; l++; }
    count += r - l + 1;               // ⭐ all subarrays ending at r and starting in [l, r]
  }
  return count;
}
// O(n) time, O(1) space.
```

`k <= 1` must be special-cased: with positive integers no product is below 1, and the loop would divide `prod` past the end. The `count += r - l + 1` idea is the same one as `count += r - l` in 3Sum Smaller — one insight, two problems.

Repeated float division is a legitimate concern here (accumulated error, and dividing by 1s never reduces the product). It's fine for LC constraints; if an interviewer pushes, offer log-sums or a prefix-product-with-binary-search alternative and note the precision trade.

### Exactly k = atMost(k) − atMost(k−1)

There's no clean single window for "exactly k distinct," because the predicate isn't monotone in the way Template B needs. The fix is subtraction.

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

Recognize this shape whenever you see "exactly." It also converts "sum equals k" into "at most k minus at most k−1" for positive arrays, and it's the standard trick for "exactly k odd numbers" (LC 1248) and similar. Two easy passes beat one clever pass you can't debug live.

### When sliding window is the wrong tool

| Problem | Why the window fails | Right tool |
|---|---|---|
| Shortest subarray with sum ≥ k, negatives allowed (LC 862) | Sum isn't monotone in window size | Prefix sums + monotonic deque |
| Subarray sum equals k, negatives allowed (LC 560) | Same | Prefix sums + hash map (Section 10) |
| Maximum sum of a **non-contiguous** subsequence | Not contiguous | Greedy / DP |
| Longest subsequence (not substring) | Not contiguous | DP (Phase 10) |
| Max in every window of size k (LC 239) | Max isn't removable in O(1) when it leaves | Monotonic deque (Phase 6) |

The last row is worth remembering: a window state must support *removal from the left in O(1)*. Sums do. Counts do. A maximum does not — removing the current max requires knowing the next-largest, which is why "sliding window maximum" needs a deque and lands in Phase 6.

---
## 10. Prefix sums

Fourth skeleton. The insight is one line:

> **A range sum is the difference of two running totals.**

Everything in this section is that sentence applied to different value types (sums, products, remainders, balances) and different dimensions.

### The 1D construction, with the offset convention that kills off-by-ones

```js
function buildPrefix(nums) {
  const pre = new Array(nums.length + 1).fill(0);   // ⭐ length n+1, pre[0] = 0
  for (let i = 0; i < nums.length; i++) pre[i + 1] = pre[i] + nums[i];
  return pre;
}
// O(n) time, O(n) space.

// Sum of nums[i..j] INCLUSIVE:
const rangeSum = (pre, i, j) => pre[j + 1] - pre[i];
```

Use the `n+1` form with `pre[0] = 0` every time. The alternative (`pre[i] = sum of nums[0..i]`) forces a special case for `i === 0` in every query, and that special case is where the bug goes. With the `n+1` convention, `pre[k]` means "sum of the first `k` elements" — no element is at a confusing index, and the empty prefix has a real slot.

Sanity check to run in your head: `rangeSum(pre, 0, n-1)` must be `pre[n] - pre[0]` = the total. If your formula doesn't produce that, you have the convention wrong.

### Build it, or stream it?

| Situation | Do this |
|---|---|
| Many range queries, array doesn't change | Build the array once: O(n) build, O(1) per query |
| One pass, you only need "sum so far" | Keep a single running variable — O(1) space |
| Array changes between queries | Prefix sums are wrong here → Fenwick/segment tree (out of scope) |

Most interview problems in this phase are the second case: you never materialize the prefix array, you keep one `sum` variable and a hash map. Building an array you only scan once is O(n) space you didn't need — an interviewer may well ask you to remove it.

### Range Sum Query — Immutable (LC 303)

```js
class NumArray {
  constructor(nums) {
    this.pre = new Array(nums.length + 1).fill(0);
    for (let i = 0; i < nums.length; i++) this.pre[i + 1] = this.pre[i] + nums[i];
  }
  sumRange(i, j) { return this.pre[j + 1] - this.pre[i]; }   // O(1)
}
// Construction O(n) time / O(n) space; each query O(1).
```

The interview framing that matters: "this trades O(n) preprocessing for O(1) queries — worth it when queries outnumber builds, which is the whole point of the immutable variant."

### Prefix sum + hash map: Subarray Sum Equals K (LC 560)

The most important problem in this section. Works with negatives, which is exactly why sliding window can't do it.

```js
function subarraySum(nums, k) {
  const seen = new Map([[0, 1]]);     // ⭐ one empty prefix, so prefixes equal to k count
  let sum = 0, count = 0;
  for (const x of nums) {
    sum += x;
    count += seen.get(sum - k) || 0;                 // check BEFORE inserting
    seen.set(sum, (seen.get(sum) || 0) + 1);
  }
  return count;
}
// O(n) time, O(n) space.
```

The derivation, which is what you say out loud: a subarray `(i..r]` sums to `k` exactly when `pre[r] − pre[i] = k`, i.e. `pre[i] = pre[r] − k`. So while scanning, ask how many earlier prefixes equal `sum − k`. Counting, not just existence — hence a `Map` of counts rather than a `Set`.

Two details that break it if you get them wrong:

- **Seed `[0, 1]`.** Without it you miss every subarray that starts at index 0.
- **Check before inserting.** With `k === 0`, inserting first lets the current prefix match itself and counts an empty subarray.

Longest-instead-of-count variant (LC 325, "Maximum Size Subarray Sum Equals k") stores the **first** index for each prefix and never overwrites:

```js
function maxSubArrayLen(nums, k) {
  const firstIdx = new Map([[0, -1]]);    // prefix 0 occurs "before" index 0
  let sum = 0, best = 0;
  for (let i = 0; i < nums.length; i++) {
    sum += nums[i];
    if (firstIdx.has(sum - k)) best = Math.max(best, i - firstIdx.get(sum - k));
    if (!firstIdx.has(sum)) firstIdx.set(sum, i);      // ⭐ keep the EARLIEST occurrence only
  }
  return best;
}
// O(n) time, O(n) space.
```

`if (!has)` before `set` is the whole trick: for a *longest* answer you want the earliest matching prefix. Overwriting gives you the shortest. Same map, opposite policy — and the mirrored rule holds: if the problem asked for the *shortest* such subarray, you'd always overwrite.

### Modular prefix: Subarray Sums Divisible by K (LC 974) — and the JS negative-`%` trap

A subarray is divisible by `k` when two prefixes share a remainder mod `k`.

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

**The trap:** `%` in JavaScript takes the sign of the dividend.

```js
-1 % 5;              // -1   (in Python: 4)
-7 % 3;              // -1
((-1 % 5) + 5) % 5;  // 4    ← the normalization you must write
```

Without the `((x % k) + k) % k` normalization, negative remainders index outside your count array (`count[-1]` is `undefined`, so `total += undefined` → `NaN`) or land in a different bucket than the equivalent positive remainder. This is a genuine JS-versus-Python difference that catches candidates who learned the pattern in Python, and it silently produces `NaN` rather than throwing. Memorize the normalization as a unit.

### The balance trick: Contiguous Array (LC 525)

"Equal numbers of 0s and 1s" becomes "prefix sum returns to a value it had before" once you map `0 → −1`.

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

The reframing — "equal counts of two things" → "signed running total revisits a value" — generalizes to any two-way balance: matched parens depth, up/down votes, gains/losses. When a problem says "equal number of X and Y," reach for ±1 and a prefix map.

### Prefix products: Product of Array Except Self (LC 238)

```js
function productExceptSelf(nums) {
  const n = nums.length;
  const res = new Array(n).fill(1);

  let prefix = 1;
  for (let i = 0; i < n; i++) { res[i] = prefix; prefix *= nums[i]; }      // product of everything left of i

  let suffix = 1;
  for (let i = n - 1; i >= 0; i--) { res[i] *= suffix; suffix *= nums[i]; } // times everything right of i

  return res;
}
// O(n) time. O(1) auxiliary space — the output array doesn't count as extra space,
// and you should say exactly that, because it's the constraint the problem is testing.
```

Two things to volunteer:

- **Why not division?** `total / nums[i]` breaks on a single zero (division by zero) and on two zeros (every answer is 0, but the total is 0 too). The problem forbids it for that reason. If asked to handle zeros with division you'd need to count them and special-case 0, 1, and ≥2 zeros — mention it, then write the prefix/suffix version.
- **Why two passes into one array?** Writing prefixes on the way up and multiplying suffixes on the way down uses the output array as the scratch space, which is how you hit O(1) auxiliary.

### 2D prefix sums (LC 304, Range Sum Query 2D)

```js
function build2D(m) {
  const rows = m.length, cols = m[0].length;
  const pre = Array.from({ length: rows + 1 }, () => new Array(cols + 1).fill(0));
  for (let i = 0; i < rows; i++)
    for (let j = 0; j < cols; j++)
      pre[i + 1][j + 1] = m[i][j] + pre[i][j + 1] + pre[i + 1][j] - pre[i][j];
  return pre;
}
// O(rows × cols) build, O(rows × cols) space.

// Sum of the rectangle (r1,c1) to (r2,c2), inclusive:
const rect = (pre, r1, c1, r2, c2) =>
  pre[r2 + 1][c2 + 1] - pre[r1][c2 + 1] - pre[r2 + 1][c1] + pre[r1][c1];
// O(1) per query.
```

Both formulas are inclusion–exclusion: subtract the strip above and the strip to the left, then add back the corner you subtracted twice. Draw the four rectangles on the whiteboard while you say it — this is one of the few places where a diagram genuinely helps, and the `+ pre[r1][c1]` term is the part people forget.

Same `+1` padding convention as 1D, for the same reason: no special-casing row 0 or column 0.

### Difference arrays — the inverse operation

Prefix sums answer many range *queries*. Difference arrays apply many range *updates* in O(1) each, then reconstruct in one pass.

```js
// Corporate Flight Bookings (LC 1109): bookings = [[first, last, seats], ...], 1-indexed
function corpFlightBookings(bookings, n) {
  const diff = new Array(n + 1).fill(0);
  for (const [first, last, seats] of bookings) {
    diff[first - 1] += seats;
    diff[last] -= seats;             // one past the end; the n+1 length makes this always in range
  }
  for (let i = 1; i < n; i++) diff[i] += diff[i - 1];
  return diff.slice(0, n);
}
// O(n + bookings) time, O(n) space.
// Naive per-booking loops would be O(n × bookings).
```

The pair `diff[l] += v; diff[r+1] -= v;` is the whole technique. Size the array `n+1` so `r+1` is never out of bounds when `r === n-1`.

This is the one pattern in the phase with a direct, non-contrived frontend use: applying hundreds of overlapping range updates (timeline/Gantt overlays, per-day availability counts, heatmap bucket accumulation) without touching each cell per update. Car Pooling (LC 1094) is the same problem with a capacity check.

---

## 11. The running-value family (Kadane and friends)

Not strictly two pointers or windows, but it lives in this phase because the signal ("maximum sum contiguous subarray") looks identical to a window problem — and it's the bridge to Phase 10's DP.

### Maximum Subarray / Kadane's (LC 53)

```js
function maxSubArray(nums) {
  let cur = nums[0], best = nums[0];        // ⚠️ initialize from nums[0], NOT 0
  for (let i = 1; i < nums.length; i++) {
    cur = Math.max(nums[i], cur + nums[i]); // start fresh at i, or extend the run ending at i-1
    best = Math.max(best, cur);
  }
  return best;
}
// O(n) time, O(1) space.
```

**The one decision per element:** is the best run ending here just `nums[i]`, or `nums[i]` plus the best run ending at `i−1`? That's a one-state DP with O(1) space — say it in DP language, because it earns the "you'll be fine in the DP round" credit early:

> "Let `cur` be the maximum sum of a subarray ending exactly at `i`. Then `cur[i] = max(nums[i], cur[i-1] + nums[i])`, and the answer is the max over all `i`. I only need the previous value, so it's O(1) space instead of an O(n) table."

**The initialization trap:** starting `best = 0` returns 0 for `[-3, -1, -2]` — wrong, the answer is −1 — because 0 implies the empty subarray is allowed. Read the problem: LC 53 requires at least one element. Initializing from `nums[0]` handles all-negative arrays automatically. Guard the empty input separately if the signature allows it.

**Why not a sliding window?** With negatives, the "shrink from the left" step is not well-defined: a prefix with negative sum should be dropped entirely, not shrunk one element at a time. Kadane's `max(nums[i], cur + nums[i])` *is* that drop, done in O(1). If all values were positive, the answer would trivially be the whole array — which tells you the problem only exists because of negatives, and therefore isn't a window problem. That reasoning is worth voicing; it shows you chose the pattern rather than pattern-matching the word "subarray."

Returning the indices as well — a common follow-up:

```js
function maxSubArrayWithRange(nums) {
  let cur = nums[0], best = nums[0], start = 0, bestL = 0, bestR = 0;
  for (let i = 1; i < nums.length; i++) {
    if (cur + nums[i] < nums[i]) { cur = nums[i]; start = i; }   // restarting → new candidate start
    else cur += nums[i];
    if (cur > best) { best = cur; bestL = start; bestR = i; }    // strict > keeps the FIRST best range
  }
  return { sum: best, from: bestL, to: bestR };
}
// O(n) time, O(1) space.
```

Note `cur > best` (strict): with `>=` you'd report the last tied range instead of the first. When a problem says "if there are ties, return the earliest," that comparison operator *is* the answer.

### Best Time to Buy and Sell Stock (LC 121) — the running-minimum form

```js
function maxProfit(prices) {
  let minSoFar = Infinity, best = 0;
  for (const p of prices) {
    minSoFar = Math.min(minSoFar, p);
    best = Math.max(best, p - minSoFar);
  }
  return best;
}
// O(n) time, O(1) space.
```

`best = 0` is correct here, and that's not inconsistent with Kadane: the problem explicitly allows doing nothing, so the empty transaction (profit 0) is a legal answer. Compare with LC 53 where the empty subarray is not legal. **Read which one the problem allows** — this is the same initialization question producing opposite answers, and interviewers do test it with an all-decreasing price array.

It's also Kadane on the difference array: the best profit is the maximum sum subarray of consecutive day-to-day deltas. Mentioning that equivalence is a nice touch and makes LC 122+ variants easier to reason about.

### Maximum Product Subarray (LC 152)

Sign flips break the single-value approach: multiplying by a negative turns the smallest product into the largest. Track both.

```js
function maxProduct(nums) {
  let mx = nums[0], mn = nums[0], best = nums[0];
  for (let i = 1; i < nums.length; i++) {
    const x = nums[i];
    const cands = [x, mx * x, mn * x];      // compute all three BEFORE reassigning
    mx = Math.max(...cands);
    mn = Math.min(...cands);
    best = Math.max(best, mx);
  }
  return best;
}
// O(n) time, O(1) space. Math.max over a fixed 3-element array is O(1).
```

The mistake to avoid: assigning `mx` first and then using the new `mx` to compute `mn`. Compute the candidates from the old values, then assign both. Zeros are handled for free — `x` itself is always a candidate, which restarts the run.

### The general shape — and the bridge to Phase 10

Every problem in this section is:

```js
let cur = <base>, best = <base>;
for (each element) {
  cur = <combine(cur, element) or restart at element>;   // best answer ENDING HERE
  best = <better of best, cur>;                          // best answer ANYWHERE
}
```

The distinction between "best ending here" and "best anywhere" is the core of 1D DP, and you're already writing it in O(1) space. When Phase 10 introduces `dp[i]` arrays, this is the same thing with the history kept. Recognizing that now makes Phase 10 dramatically shorter.

---

## 12. Index-as-hash and cyclic sort — O(1)-space array tricks

Small family, appears often, and it's the one place where "O(1) extra space" is achievable for problems that look like they need a hash set. The enabling condition is always the same: **values are constrained to the range 1..n (or 0..n) where n is the array length**, so the array can be its own hash table.

Be honest with yourself about these: they are trick-dependent. Know them; don't expect to derive them live.

### Missing Number (LC 268) — values 0..n, one missing

```js
// Sum formula
function missingNumber(nums) {
  const n = nums.length;
  const expected = (n * (n + 1)) / 2;
  let actual = 0;
  for (const x of nums) actual += x;
  return expected - actual;
}
// O(n) time, O(1) space. Safe for n up to ~2^26 before Number precision matters.

// XOR — no overflow concern at all, and the version to prefer if asked about large n
function missingNumberXor(nums) {
  let x = nums.length;
  for (let i = 0; i < nums.length; i++) x ^= i ^ nums[i];
  return x;
}
// O(n) time, O(1) space.
```

JS numbers are IEEE-754 doubles: exact integers up to 2^53, so the sum formula won't silently overflow the way a 32-bit `int` would in Java or C++. Worth one sentence — it's a real cross-language difference and shows you know why the XOR variant exists elsewhere. Note that `^` coerces to 32-bit signed integers, which is fine here but is its own trap for large values.

### Find All Numbers Disappeared in an Array (LC 448) — negation marking

Values are 1..n; mark presence by negating the value at the corresponding index.

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

Always read the value through `Math.abs` — after the first pass some entries are already negative, and forgetting that gives you a negative index. **Find All Duplicates (LC 442)** is the same loop, collecting `Math.abs(x)` when the target slot is already negative.

The honest caveat to state: this mutates the input, which is only acceptable if the problem allows it. If it doesn't, you're back to a `Set` and O(n) space. Volunteering that trade is better than being caught by it.

### First Missing Positive (LC 41) — cyclic sort

Hard problem, standard trick: put each value `v` in `1..n` at index `v−1`, then scan.

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

Two things to be able to defend:

- **Why the nested `while` is still O(n):** every swap places at least one value in its final correct slot, and a value never leaves a correct slot. So there are at most n swaps in total across the entire outer loop. Same amortized argument as sliding window — resets versus persists.
- **Why the loop condition tests `nums[nums[i]-1] !== nums[i]` rather than `i !== nums[i]-1`:** the value-based test also terminates on duplicates. With the index-based test, `[1, 1]` swaps forever.

### Find the Duplicate Number (LC 287)

Values in `1..n`, exactly one duplicate, must not modify the array, O(1) space. The intended solution is Floyd's cycle detection treating the array as a linked list (`i → nums[i]`) — which belongs to Phase 5. Recognize it now, implement it there. If you're asked it before Phase 5, the binary-search-on-value-range answer is O(n log n) and fully acceptable:

```js
function findDuplicate(nums) {
  let lo = 1, hi = nums.length - 1;
  while (lo < hi) {
    const mid = (lo + hi) >> 1;
    let count = 0;
    for (const x of nums) if (x <= mid) count++;
    if (count > mid) hi = mid;      // too many values ≤ mid → duplicate is in [lo, mid]
    else lo = mid + 1;
  }
  return lo;
}
// O(n log n) time, O(1) space. Binary search on the answer space — previews Phase 3, Day 18.
```

---
## 13. JS/V8 gotchas that catch candidates in this phase

Phase 0 covered the general cost traps. These are the ones that specifically bite on array/string/two-pointer problems.

### 1. `Math.max(...arr)` throws on large arrays

```js
Math.max(...bigArray);   // ❌ RangeError: Maximum call stack size exceeded
```

Spread becomes function arguments, and the argument-count limit in V8 is on the order of tens of thousands (roughly 65k–125k depending on version and stack headroom). It works in every test you run locally with small inputs and blows up on the real one.

```js
let mx = -Infinity;
for (const x of arr) if (x > mx) mx = x;      // ✅ O(n), no limit
arr.reduce((a, b) => Math.max(a, b), -Infinity);   // ✅ also fine
```

**But** `Math.max(...count)` on a fixed 26-element array (Section 9) is perfectly safe and O(1) — the array length is a compile-time constant. The rule is about arrays whose length scales with the input. Being able to draw that line is more impressive than blanket-avoiding spread.

### 2. `sort()` mutates, and string-compares by default

```js
[10, 9, 1].sort();                  // [1, 10, 9]   ← lexicographic
[10, 9, 1].sort((a, b) => a - b);   // [1, 9, 10]   ✅
```

Every k-sum solution in Section 7 mutates the caller's array. Say it: *"this sorts in place, so it mutates the input — if the caller needs the original order I'll copy first, which costs O(n) space."* On LeetCode nobody cares; in a code review it's a bug, and interviewers from product teams notice.

Descending: `(a, b) => b - a`. Never `(a, b) => a > b` — a boolean comparator is not a valid comparator (it coerces to 0/1 and never returns −1), so it produces subtly wrong orders.

### 3. `%` is remainder, not modulo

Covered in Section 10, repeated because it silently yields `NaN`:

```js
((sum % k) + k) % k     // the only form to write when the value can be negative
```

### 4. `indexOf` vs `includes` on `NaN` and `-0`

```js
[NaN].indexOf(NaN);     // -1     (strict equality: NaN !== NaN)
[NaN].includes(NaN);    // true   (SameValueZero)
[-0].includes(0);       // true
Object.is(-0, 0);       // false
```

Rarely decisive, but if a problem involves `NaN` sentinels, `includes` is the one that behaves as you'd expect. Both are still O(n) — never in a loop.

### 5. No integer overflow, so skip the overflow-safe midpoint ritual

```js
const mid = Math.floor((lo + hi) / 2);   // fine in JS for any realistic n
const mid = lo + Math.floor((hi - lo) / 2);   // the Java/C++ habit — harmless, unnecessary here
const mid = (lo + hi) >> 1;              // fast, but silently breaks above 2^31
```

Exact integers hold to 2^53, so `lo + hi` cannot overflow for any array you could allocate. If you write the `lo + (hi - lo) / 2` form anyway, say you know it's belt-and-braces in JS — otherwise it reads as cargo-culted from another language. Watch the `>> 1` form: bitwise operators coerce to 32-bit signed, so it breaks past ~2.1 billion. Fine for arrays, wrong for binary search over a large *value* range (Phase 3, Day 18).

### 6. Strings: `str[i]` is O(1), `slice` is O(n)

```js
s[i]              // O(1) — use this in two-pointer and window loops
s.charAt(i)       // O(1) — same thing, older style
s.charCodeAt(i)   // O(1) — use for the 26-bucket count trick
s.slice(a, b)     // O(n) — NEVER inside a window loop
s.split('')       // O(n) allocation, and breaks surrogate pairs
```

The counting idiom, worth having in muscle memory:

```js
const idx = c => c.charCodeAt(0) - 97;              // 'a' → 0, lowercase only
const counts = new Array(26).fill(0);
```

If the problem says "letters" without specifying case, ask. If it says any ASCII, use `new Array(128).fill(0)` or a `Map`.

### 7. Unicode: `length` lies, and it can break a palindrome check

```js
'👍'.length;             // 2   — one code point, two UTF-16 units
[...'👍'].length;        // 1   — spread iterates code points
'café'.length;           // 4, but 'café'.length is 5 — same glyph, different length

// Reversing with split breaks emoji:
'ab👍'.split('').reverse().join('');   // mojibake — the surrogate pair is torn apart
[...'ab👍'].reverse().join('');        // '👍ba' ✅
```

For LeetCode-style inputs (ASCII) none of this matters. In an interview, one sentence — *"I'm assuming ASCII; with real user input I'd iterate code points via `[...str]`, or use `Intl.Segmenter` for grapheme clusters"* — is exactly the kind of frontend-flavored depth that lands, because it's a bug they've actually shipped.

### 8. Comparing floats after repeated division

Section 9's product window divides repeatedly. `0.1 * 3 !== 0.3`. For LC constraints it passes; if a problem involves fractional products, mention log-sums or exact integer arithmetic rather than pretending float division is exact.

### 9. `arr.length` inside the loop condition

```js
for (let i = 0; i < arr.length; i++) { arr.push(x); }   // ❌ infinite loop
```

`arr.length` is re-read each iteration. If the loop body mutates length (push/pop/splice), cache it: `const n = arr.length`. This bites in write-pointer problems where you're tempted to `pop` the tail.

### 10. `undefined` in arithmetic gives `NaN`, silently

```js
const arr = [1, 2, 3];
arr[5] + 1;             // NaN — not an error
sum += arr[i];          // one out-of-bounds read poisons everything downstream
```

Java throws, Python throws, JS returns `undefined` and then `NaN`. A single off-by-one gives you `NaN` fifty lines later with no stack trace. **If your answer is `NaN`, you have an out-of-bounds read** — that's the fastest debugging heuristic in this phase.

### 11. Object keys are strings; `Map` keys are not

```js
const o = {};
o[1] = 'a'; o['1'] = 'b';
o;                        // { '1': 'b' }  — one key, coerced

const m = new Map();
m.set(1, 'a').set('1', 'b');
m.size;                   // 2  ✅ distinct keys
```

Use `Map` for window counters whenever keys are numbers, and always for the `size` property (O(1) versus `Object.keys(o).length` at O(n)). The `-0`/`0` and `NaN` key behaviors of `Map` follow SameValueZero, which is the sane choice.

### 12. `slice` inside recursion or loops

Already flagged, restated because it's the top source of accidental O(n²) in this phase: pass indices, not sliced copies. `helper(arr, i)` not `helper(arr.slice(i))`.

---

## 14. Complexity reference for every pattern here

| Pattern | Time | Space | Notes |
|---|---|---|---|
| Write pointer (in place) | O(n) | O(1) | one read pass, ≤ n writes |
| Two pointers, opposite ends | O(n) | O(1) | needs sorted input (or symmetry) |
| Two pointers after sorting | O(n log n) | O(1)–O(n) | sort dominates; sort space is engine-dependent |
| 3Sum | O(n²) | O(1) aux | n × two-pointer scan |
| 4Sum / k-Sum | O(n^(k−1)) | O(1) aux | (k−2) nested loops + scan |
| Fixed sliding window | O(n) | O(1) or O(k) | O(k) only if the state is a map/set |
| Variable sliding window | O(n) | O(1)–O(charset) | pointer movement ≤ 2n |
| Window with `Map` counter | O(n) | O(k) | `map.size` is O(1) — use it |
| Window with 26/128 array | O(n) | O(1) | fixed alphabet is a constant |
| Exactly-k via atMost(k)−atMost(k−1) | O(n) | O(k) | two linear passes |
| Prefix sum array | O(n) build, O(1) query | O(n) | worth it for many queries |
| Prefix sum streamed | O(n) | O(1) | when you need only "so far" |
| Prefix sum + hash map | O(n) | O(n) | handles negatives; window doesn't |
| 2D prefix sum | O(mn) build, O(1) query | O(mn) | inclusion–exclusion |
| Difference array | O(n + u) | O(n) | u = number of range updates |
| Kadane / running value | O(n) | O(1) | one-state DP |
| Negation marking | O(n) | O(1) aux | mutates the input |
| Cyclic sort | O(n) | O(1) | ≤ n swaps total, amortized |
| Brute force pairs | O(n²) | O(1) | the baseline you name and then beat |
| Brute force all subarrays | O(n²) or O(n³) | O(1) | O(n³) if you re-sum each subarray |

Two lines worth internalizing:

- **All subarrays is O(n²)** — there are n(n+1)/2 of them. If you also re-sum each one, it's O(n³). Naming "O(n³) brute force, O(n²) with a running sum, O(n) with a window" as a three-step ladder is a strong opening in any subarray problem.
- **Sorting to enable two pointers costs O(n log n)** — so if a hash map gets you O(n), the hash map wins. Sort when you need *ordering* (dedup, triplets, ranges), hash when you need *lookup*.

---

## 15. Frontend relevance — the honest version

Some of these are real. Some connections you'll see claimed online are forced, and I'll say which.

**Sliding window → genuinely everywhere in frontend.**
- Rate limiting and analytics: "how many events in the last 60 seconds" is a variable window over a timestamp array. This is exactly how client-side throttling of telemetry batches is built.
- Moving averages for smoothing: FPS counters, scroll velocity, sensor/pointer data. A fixed window with `sum += arr[i] - arr[i-k]` is the implementation.
- Virtualized lists: the set of rendered rows *is* a window over the data array, and `startIndex`/`endIndex` are `l` and `r`. react-window and TanStack Virtual maintain exactly this.

**Prefix sums → variable-height virtualization.** This is the strongest real example in the phase. To render a virtualized list where rows have different heights, you keep a prefix sum of heights; finding the first visible row for a given `scrollTop` is a binary search over that prefix array (O(log n) instead of O(n) per scroll event). Building it is O(n) once, updating a row's height invalidates the suffix. If you've used TanStack Virtual's dynamic measurement, that's the data structure underneath.

**Difference arrays → overlapping range updates.** Availability calendars, Gantt/timeline overlays, "how many bookings overlap each day." Applying u updates naively is O(n·u); the difference array makes it O(n + u).

**Two pointers → merging two sorted streams.** Reconciling a sorted server page with a sorted local cache, diffing two sorted ID lists to compute added/removed sets, merging sorted results from two endpoints. This is a real thing you write in a data layer, and it's why `merge` in Section 6 is worth knowing cold.

**Write pointer → mostly interview-only in React, and you should say so.** In React you don't mutate state arrays; you produce new ones with `filter`/`map` because reference identity drives re-renders. The in-place write pointer is the wrong tool inside a component. Where it *is* right: hot non-React code — canvas/WebGL particle buffers, audio processing, large data transforms in a worker where allocation pressure and GC pauses matter. If an interviewer asks "would you write this in your React app?", the correct answer is "no — immutability wins there; this matters in a worker or a canvas loop where allocations cause frame drops." That answer is better than pretending the trick is universally applicable.

**Kadane → weakest link.** Occasionally useful for "best streak" metrics in a dashboard. Don't oversell it; the honest pitch is that it's the O(1)-space DP pattern that Phase 10 builds on.

**The DOM angle** for `moveZeroesSwap`'s `if (w !== r)` guard: skipping no-op swaps matters concretely when the "swap" is `insertBefore` on real nodes, because every unnecessary DOM move costs layout work. That's a legitimate bridge from the algorithm to browser performance.

---

## 16. Interview scripts, per pattern

Phase 0 gave you the general open/close. These are the pattern-specific lines. Say the *reason*, not just the name — "two pointers" is a label, "sortedness lets me discard a whole side per comparison" is understanding.

**Opening any array problem:**
> "Brute force is checking every pair, O(n²). The array is sorted, though — that means comparing the two ends tells me which end can't be part of the answer, so I can converge two pointers in O(n) with O(1) space. Want me to go straight there?"

**Two pointers, opposite ends:**
> "I'm moving the pointer that can't improve the answer. Since `nums[l] + nums[r]` is below the target and everything left of `r` is smaller, `nums[l]` can't pair with anything in range — so `l` is done permanently. Each pointer only moves inward, at most n moves total, so O(n)."

**Container With Most Water specifically:**
> "The area is capped by the shorter wall, so any other container using that wall has less width and no more height — it can never beat what I just recorded. That makes discarding the shorter side safe."

**Write pointer:**
> "I'll keep a write index trailing a read index. The invariant is that `arr[0..w-1]` is always the final answer so far, and `w <= r` always, so I never overwrite something I still need. O(n) time, O(1) space, and it returns the new length as asked."

**Sliding window — and this is the one to have word-perfect:**
> "This is a contiguous-subarray problem, and validity is monotone: adding on the right can only make it worse, removing from the left can only make it better. So I'll expand right every step and shrink left while the window is invalid, recording the best after each shrink. It looks like a nested loop but it's O(n) — `left` never resets, it only moves forward, at most n times across the whole run. Total pointer movement is bounded by 2n."

**Window, when they ask about the nested loop:**
> "The inner `while` doesn't multiply, it adds. If the inner counter reset each outer iteration it'd be O(n²); it persists, so total inner work is bounded by n across all iterations. Amortized O(1) per step."

**Window, showing you know its limits — high-value, most candidates never say it:**
> "One caveat: this relies on all values being positive. With negatives, adding an element can decrease the sum, so shrinking is no longer guaranteed to help and the window breaks — I'd switch to prefix sums with a hash map, or a monotonic deque if I needed the shortest such subarray."

**Prefix sum:**
> "A range sum is the difference of two running totals, so I'll keep a running sum and a map from prefix value to how many times I've seen it. A subarray sums to k exactly when a previous prefix equals `sum − k`. O(n) time, O(n) space, and unlike a sliding window this handles negative numbers."

**Kadane:**
> "One-state DP: `cur` is the best subarray sum ending exactly here, either this element alone or this element extending the previous run. The answer is the max over all positions. I only need the previous value, so O(1) space instead of an O(n) table."

**Closing any solution — always volunteer space, and always mention mutation:**
> "Time is O(n), space is O(1) auxiliary. One note: this sorts/mutates the input array — if the caller needs it preserved I'd copy first, which makes it O(n) space."

**When you're stuck — say the search, don't go quiet:**
> "Let me check the structure I can exploit. It's not sorted, so opposite-end pointers are out unless I pay O(n log n) to sort — and sorting destroys the indices, which the output needs. The answer is contiguous, so a window is plausible, but there are negative numbers, so validity isn't monotone. That points at prefix sums plus a hash map."

That last script is the most valuable thing in this section. Interviewers hire the person who narrates a structured search over the person who silently produces the memorized answer.

---

## 17. Off-by-one discipline and the dry-run protocol

Every failure in this phase is one of six things. Check them in this order before you say "I think that's right."

1. **Window length.** `r - l + 1`, not `r - l`. Verify with `l === r`: one element, length 1.
2. **Loop bound.** For windows of size k by start index: `i + k <= n`. For adjacent pairs: `i < n - 1`. For last-element access: `i <= n - 1`.
3. **Which element leaves.** In a fixed window at step `i`, the departing element is `arr[i - k]`.
4. **Where you record the answer.** Longest → after the shrink loop. Shortest → inside it. Getting this backwards passes the sample test and fails the hidden ones.
5. **Initialization.** `best = 0` when the empty answer is legal; `best = -Infinity`/`nums[0]` when it isn't; `best = Infinity` for minimize.
6. **Strictness.** `l < r` for distinct pairs, `l <= r` when the middle element must be processed. `>` versus `>=` when ties must resolve to the first occurrence.

### The dry-run protocol

Before you run anything, trace these five inputs on paper. It takes 90 seconds and catches nearly everything.

| Input | What it catches |
|---|---|
| `[]` | boundary reads, `reduce` without an initial value, `length - 1` going negative |
| `[5]` | `l < r` never entering the loop, `i = 1` loops never running |
| `[2, 2]` or `"aa"` | duplicate handling, shrink-before-add ordering |
| all negative, e.g. `[-3, -1, -2]` | initialization traps (Kadane, min/max seeds) |
| the smallest input where the answer is NOT the whole array | recording the answer in the wrong place |

Write the trace as a table of `l, r, state, best` per step. Interviewers explicitly value seeing this — Phase 0's rule stands: debugging by clicking Run is a red flag in a live interview.

### Two sanity checks that catch most wrong answers instantly

- **Answer is `NaN`** → out-of-bounds read (Section 13, gotcha 10) or an unnormalized negative `%`.
- **Answer is exactly the array length, or exactly 0, on a case where it shouldn't be** → the recording position or the initialization is wrong, not the logic.

---

## 18. The 7-day plan with problem assignments

Days 3–9, following the roadmap. Guided means you already know the pattern applies; blind means you decide. Every problem below is a LeetCode number.

### Day 3 — Traversal, in-place, write pointer

Read: Sections 3, 4, 5.

| Type | Problems |
|---|---|
| Guided | 27 Remove Element · 26 Remove Duplicates from Sorted Array |
| Blind | 283 Move Zeroes · 80 Remove Duplicates from Sorted Array II |
| Stretch | 88 Merge Sorted Array (backward fill) |

Target by end of day: write the write-pointer template from memory and state both invariants out loud.

### Day 4 — Two pointers, opposite ends

Read: Section 6.

| Type | Problems |
|---|---|
| Guided | 344 Reverse String · 167 Two Sum II |
| Blind | 125 Valid Palindrome · 11 Container With Most Water |
| Stretch | 42 Trapping Rain Water (write the O(n)-space version first, then compress) |

Target: give the exchange argument for Container With Most Water unprompted.

### Day 5 — Same-direction pointers, sort + two pointers

Read: Section 7.

| Type | Problems |
|---|---|
| Guided | 977 Squares of a Sorted Array · 75 Sort Colors |
| Blind | 15 3Sum · 16 3Sum Closest |
| Stretch | 18 4Sum · 259 3Sum Smaller |

Target: 3Sum with all three duplicate skips correct, from memory. This is the hardest single day in the phase — if you only half-get 3Sum, redo it on Day 9 rather than moving on with a shaky version.

### Day 6 — Fixed sliding window

Read: Section 8.

| Type | Problems |
|---|---|
| Guided | 643 Maximum Average Subarray I · "max sum subarray of size k" (classic, not on LC) |
| Blind | 438 Find All Anagrams in a String · 567 Permutation in String |
| Stretch | 1456 Maximum Number of Vowels in a Substring of Given Length |

Target: `sum += arr[i] - arr[i-k]` without hesitation, and the `i >= k - 1` recording gate explained.

### Day 7 — Variable sliding window

Read: Sections 8 and 9. This is the highest-value day in the phase.

| Type | Problems |
|---|---|
| Guided | 3 Longest Substring Without Repeating Characters · 209 Minimum Size Subarray Sum |
| Blind | 424 Longest Repeating Character Replacement · 1004 Max Consecutive Ones III · 904 Fruit Into Baskets |
| Stretch | 76 Minimum Window Substring · 713 Subarray Product Less Than K |

Target: write both Template B and Template C from memory and say which one a new problem needs within 15 seconds of reading it.

### Day 8 — Prefix sums and running values

Read: Sections 10 and 11.

| Type | Problems |
|---|---|
| Guided | 303 Range Sum Query Immutable · 560 Subarray Sum Equals K |
| Blind | 238 Product of Array Except Self · 53 Maximum Subarray · 525 Contiguous Array |
| Stretch | 974 Subarray Sums Divisible by K · 304 Range Sum Query 2D · 1109 Corporate Flight Bookings |

Target: explain why 560 needs prefix sums instead of a sliding window, and write the `((x % k) + k) % k` normalization without prompting.

### Day 9 — Mixed practice, no hints

Read: Sections 2, 16, 17, and the cheat sheet.

Work the Section 19 practice set cold. For each problem, before writing anything: name the pattern, state the target time and space, then write the skeleton. Add these if you have time:

| Type | Problems |
|---|---|
| Mixed blind | 121 Best Time to Buy and Sell Stock · 152 Maximum Product Subarray · 448 Find All Numbers Disappeared · 992 Subarrays with K Different Integers |
| Re-solve | Anything from Days 3–8 you needed help on — and the 3-day/10-day re-solve rule starts now |

Target: 60-second pattern identification on unseen problems, with the target complexity stated before any code.

---
## 19. Practice — 32 problems

Answers in Section 20. Work them cover-down.

**Protocol for each one, in this order:**

1. Name the pattern (from Section 2's table).
2. State the target time and space **before** writing code.
3. Write the skeleton from memory.
4. Dry-run the five inputs from Section 17.
5. Only then check the answer.

Problems 1–12 are deliberate recall — they're solved in the body above, so if you can't reproduce them cold, the body didn't stick. Problems 13–32 are variants that are *not* worked in the body; they're the real test of whether you own the patterns or just recognize them.

### Recall set (solved above — reproduce from memory)

**1.** (LC 80) Remove duplicates from a sorted array so each value appears **at most twice**, in place. Return the new length. Generalize to at most `k`.

**2.** (LC 283) Move all zeroes to the end of an array in place, preserving the relative order of the non-zeroes. Minimize the number of writes.

**3.** (LC 88) `nums1` has `m` sorted values followed by `n` empty slots; `nums2` has `n` sorted values. Merge into `nums1` in place. O(1) extra space.

**4.** (LC 125) Return whether a string is a palindrome considering only alphanumeric characters, case-insensitive. **O(1) extra space** — no cleaned copy.

**5.** (LC 167) Given a **sorted** array and a target, return the 1-indexed positions of the two numbers that sum to the target. O(1) space.

**6.** (LC 11) Given `height[]`, pick two lines forming a container with the x-axis holding the most water. Return the max area. Give the argument for why your pointer move is safe.

**7.** (LC 75) Sort an array containing only 0s, 1s and 2s in place, **one pass**, O(1) space.

**8.** (LC 15) Return all **unique** triplets summing to zero.

**9.** (LC 3) Length of the longest substring with no repeating characters.

**10.** (LC 209) Minimal length of a contiguous subarray with sum ≥ target. Return 0 if none. All values positive.

**11.** (LC 560) Count the subarrays summing to exactly `k`. **Values may be negative.**

**12.** (LC 238) Return an array where `res[i]` is the product of every element except `nums[i]`. No division, O(1) extra space beyond the output.

### Variant set (not solved above)

**13.** (LC 345) Reverse only the vowels of a string, leaving all other characters in place. `"leetcode"` → `"leotcede"`.

**14.** (LC 392) Given `s` and `t`, return whether `s` is a subsequence of `t`. Follow-up: what changes if you get 10⁹ different `s` values against one fixed `t`?

**15.** (LC 844) Compare two strings where `#` means backspace. `"ab#c"` and `"ad#c"` are equal. Do it in **O(1) space** — no string building.

**16.** (LC 680) Return whether a string can be made a palindrome by deleting **at most one** character.

**17.** (LC 881) Each boat carries at most two people and at most `limit` total weight. Given `people[]` weights, return the minimum number of boats. (Every person's weight ≤ limit.)

**18.** (LC 581) Find the length of the shortest continuous subarray that, if sorted, makes the whole array sorted. Target O(n) time, O(1) space.

**19.** (LC 845) Return the length of the longest "mountain" — a strictly increasing run followed by a strictly decreasing run, both non-empty. O(1) space.

**20.** (LC 1299) Replace every element with the greatest element to its right; the last becomes −1. In place.

**21.** Classic (not on LC): given an array and `k`, return the **maximum sum of any contiguous subarray of size exactly k**. Handle `k > n`.

**22.** (LC 1456) Return the maximum number of vowels in any substring of length `k`.

**23.** (LC 1343) Count the subarrays of size `k` whose **average** is ≥ `threshold`. Avoid floating-point division.

**24.** (LC 2090) For each index, return the average of the subarray of radius `k` centered there (window size `2k+1`), or −1 where the full window doesn't fit. Integer division, truncated.

**25.** (LC 1052) `customers[i]` arrive at minute `i`; `grumpy[i]` is 1 if the owner is grumpy then (those customers leave unsatisfied). The owner may be non-grumpy for one window of `minutes` consecutive minutes. Return the max satisfied customers.

**26.** (LC 904) You can carry only two types of fruit. Walking left to right through `fruits[]`, return the maximum number of fruits you can collect from a contiguous run. (Generalize: at most `k` distinct.)

**27.** (LC 1004) Given a binary array and `k`, return the length of the longest run of 1s if you may flip at most `k` zeroes.

**28.** (LC 76) Minimum window substring of `s` containing all characters of `t`, including duplicates. Return `""` if none.

**29.** (LC 713) Count the contiguous subarrays whose product is strictly less than `k`. All values ≥ 1.

**30.** (LC 795) Count the contiguous subarrays whose **maximum** element is in `[left, right]`.

**31.** (LC 696) Count the substrings with equal numbers of consecutive 0s and 1s, where all the 0s and all the 1s are grouped (`"00110011"` → 6). Careful: this looks like a window problem and isn't.

**32.** (LC 41) Find the smallest missing **positive** integer. O(n) time, O(1) space.

### Self-check before you look at the answers

- Which of these 32 are *not* window problems despite mentioning subarrays or substrings? (Answer: 11, 12, 18, 20, 30 partially, 31, 32 — and knowing why matters more than solving them.)
- Which three need a sort first, and what does that cost you? (8, 17, and any k-sum — O(n log n), plus loss of original indices.)
- Which mutate their input, and would you flag that in a code review? (1, 2, 3, 7, 20, 32, plus 8 and 17 via `sort`.)

---
## 20. Worked answers

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
Check: `nums[w-k]` looks back into the *output*, not the input. If you wrote `nums[r-k]` you got lucky on some tests and wrong on `[1,1,1,2,2,3]`.

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
// O(n) time, O(1) space. Writes only when a non-zero actually has to move.
```
Check: the `w !== r` guard. Without it you do n self-swaps — correct output, unnecessary writes, and the interviewer asked for minimum writes.

**3. Merge Sorted Array (LC 88)** — two pointers, backward fill.

```js
function merge(nums1, m, nums2, n) {
  let i = m - 1, j = n - 1, w = m + n - 1;
  while (j >= 0) nums1[w--] = (i >= 0 && nums1[i] > nums2[j]) ? nums1[i--] : nums2[j--];
}
// O(m + n) time, O(1) space.
```
Check: loop on `j >= 0` only. Leftover `nums1` values are already in position, so copying them is wasted work.

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
Check: `l < r` inside both inner loops. Test `",."` and `"a."`.

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
Check: could you say *why* discarding is safe? "`nums[l]` paired with anything ≤ `nums[r]` is still too small, so `l` is eliminated permanently."

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
Check: the exchange argument, in one sentence, without prompting.

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
Check: no `mid++` in the `=== 2` branch, and `mid <= high` not `mid < high`. Test `[2,0,1]` and `[2,2]`.

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
Check: three skips, `i` compared **backward** to `i-1`, and the `l`/`r` skips only inside the hit branch. Test `[0,0,0,0]` → exactly one triplet.

**9. Longest Substring Without Repeating Characters (LC 3)** — Template B with a Set.

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
// O(n) time, O(min(n, charset)) space.
```
Check: shrink before adding. Test `"aa"` (2 → answer 1) and `"abba"` (answer 2).

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
Check: record **inside** the shrink loop (minimize), and convert `Infinity → 0`. Also say the positivity precondition out loud.

**11. Subarray Sum Equals K (LC 560)** — prefix sum + hash map, because negatives rule out a window.

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
Check: seed `[0,1]`, and query before insert. Test `[1,-1,0]` with `k=0` → 3.

**12. Product of Array Except Self (LC 238)** — prefix and suffix products in the output array.

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
Check: `res[i] = p` **before** updating `p`, both directions. Test `[0,0]` → `[0,0]` and `[1,0]` → `[0,1]`.

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
// O(n) time, O(n) space (JS strings are immutable — there is no O(1)-space option).
```
Volunteer that last point: in C++ this would be O(1) space; in JS you must materialize an array. That's a real language difference, not a weakness in your solution. Using a `Set` for the vowels is fine too and arguably cleaner — `includes` on a fixed 10-char string is O(1) because the length is constant.

**14. Is Subsequence (LC 392)** — same-direction pointers over two sequences.

```js
function isSubsequence(s, t) {
  let i = 0;
  for (const c of t) if (i < s.length && c === s[i]) i++;
  return i === s.length;
}
// O(|t|) time, O(1) space.
```
The follow-up is the actual interview question. For many `s` against one fixed `t`, preprocess `t` into `Map<char, sortedIndices[]>` once in O(|t|), then per query binary-search the next occurrence after the current position: O(|s| log |t|) per query. That's the pattern shift they're testing — from scanning to indexing — and it previews Phase 3.

**15. Backspace String Compare (LC 844)** — pointers from the back, counting pending deletions.

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
Why backward: a `#` affects the character *before* it, so scanning right-to-left lets you resolve deletions without a buffer. The O(n)-space version (build both strings with a stack/array, then compare) is easier and worth writing first if you're shaky — then compress. Test `"a##c"` vs `"#a#c"` → true, and `"ab##"` vs `"c#d#"` → true.

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
Still O(n), not O(n²): the branch happens at most once, and each helper call scans a disjoint remainder. Say that — it's the question the interviewer will ask. Extending to "at most k deletions" is *not* a two-pointer problem any more; it becomes edit-distance DP (Phase 10), and recognizing that boundary is the follow-up.
**17. Boats to Save People (LC 881)** — sort, then greedy opposite-end pointers.

```js
function numRescueBoats(people, limit) {
  people.sort((a, b) => a - b);
  let l = 0, r = people.length - 1, boats = 0;
  while (l <= r) {                       // ⚠️ <= : the last person still needs a boat
    if (people[l] + people[r] <= limit) l++;   // pair the lightest with the heaviest
    r--;                                       // the heaviest always boards, alone or paired
    boats++;
  }
  return boats;
}
// O(n log n) time (sort dominates), O(1) auxiliary space.
```
Two things here. **`l <= r`, not `l < r`** — this is the one problem in the set where a single middle element must still be processed, because a lone person occupies a boat. And the greedy needs a justification: the heaviest person must board some boat, and the best possible companion is the lightest remaining — if even that doesn't fit, no one fits, so they go alone. Being able to state the greedy exchange argument is what separates this from guessing.

**18. Shortest Unsorted Continuous Subarray (LC 581)** — two directional scans tracking running extremes.

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
The idea: the last index that is smaller than some earlier element must be inside the window, and symmetrically from the right. The obvious solution — sort a copy and compare — is O(n log n) time and O(n) space; present that first, then this. `end === -1` means already sorted → return 0.

**19. Longest Mountain in Array (LC 845)** — a single scan consuming up-runs and down-runs.

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
Both runs must be non-empty — that's what "mountain" requires, and the `up > 0 && down > 0` guard is the whole problem. The nested `while`s are still O(n) total because `i` never resets: same amortized argument as sliding window. Test `[2,2,2]` → 0 and `[0,1,0]` → 3.

**20. Replace Elements with Greatest Element on Right (LC 1299)** — suffix maximum, right to left.

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
Read the old value into a temp *before* overwriting. This is the "suffix aggregate" half of Product Except Self — same right-to-left shape, different operator. The brute force is O(n²); naming the suffix scan is the point.

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
This is the template everything else in Day 6 is built on. Recomputing each window's sum would be O(n·k). Guard `k > n` explicitly rather than returning `-Infinity`.

**22. Maximum Number of Vowels in a Substring of Length k (LC 1456)** — fixed window with a counter.

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
    if (best === k) return k;              // can't beat a full window — early exit
  }
  return best;
}
// O(n) time, O(1) space.
```
The early return is a free constant-factor win worth mentioning: once you hit `k`, no later window can be better. Explicit `||` comparisons beat `'aeiou'.includes(c)` in a hot loop; both are O(1), so say it's a constant-factor choice, not a complexity one.

**23. Subarrays of Size k with Average ≥ Threshold (LC 1343)** — fixed window, compare against `k * threshold`.

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
Comparing `sum >= k * threshold` instead of `sum / k >= threshold` removes n divisions and all floating-point error. Same trick as deferring the division in Maximum Average Subarray.

**24. K-Radius Subarray Averages (LC 2090)** — fixed window of size `2k+1`, centered.

```js
function getAverages(nums, k) {
  const n = nums.length, size = 2 * k + 1, res = new Array(n).fill(-1);
  if (size > n) return res;
  let sum = 0;
  for (let i = 0; i < size; i++) sum += nums[i];
  res[k] = Math.floor(sum / size);
  for (let i = size; i < n; i++) {
    sum += nums[i] - nums[i - size];
    res[i - k] = Math.floor(sum / size);      // center of the window ending at i
  }
  return res;
}
// O(n) time, O(n) output space, O(1) auxiliary.
```
Three traps: window size is `2k+1` (not `2k`), the result index is the window's **center** (`i - k`), and `k = 0` must return the array itself (`size = 1` — it does, for free). Worth one JS-specific line: sums here reach ~10¹⁰, which is exact in a double (safe to 2⁵³), so unlike Java there's no overflow concern and no `long` needed. `Math.floor` is correct because the problem specifies truncation and all values are non-negative — if negatives were allowed, `Math.trunc` and `Math.floor` would differ and truncation would be the one they mean.

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
The reframing is the whole problem: customers who are already satisfied are a constant, so the window should maximize only the *additional* customers the secret technique saves. Sliding a window over "total customers" gives a wrong answer that looks reasonable. When a problem has a fixed baseline plus a windowed bonus, separate them explicitly — that habit generalizes.

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
Delete at zero, or `count.size` counts departed types forever and the constraint never trips. Parameterizing `k` instead of hardcoding 2 answers the generalization follow-up before it's asked.

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
Note that `l` and `zeros` never reset — the `while` runs at most n times overall. This and 424 and 904 are the same skeleton with three different notions of "invalid"; if that isn't obvious to you yet, write all three back to back.

**28. Minimum Window Substring (LC 76)** — Template C with `need`/`window`/`formed`.

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
// O(|s| + |t|) time, O(|s| + |t|) space.
```
`===` when incrementing `formed`, `<` when decrementing, and one `slice` at the very end. Test `s="a", t="aa"` → `""` (duplicates in `t` matter) and `s="ab", t="b"` → `"b"`.

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
`count += r - l + 1` counts every subarray ending at `r`. The `k <= 1` guard prevents `l` from running past `r` (no positive-integer product is below 1). Say the precondition: all values ≥ 1, otherwise a zero or a fraction breaks the monotonicity that lets you shrink.

**30. Number of Subarrays with Bounded Maximum (LC 795)** — the subtraction trick again, without a map.

```js
function numSubarrayBoundedMax(nums, left, right) {
  const atMost = (bound) => {
    let count = 0, run = 0;
    for (const x of nums) {
      run = x <= bound ? run + 1 : 0;   // subarrays ending here with all elements ≤ bound
      count += run;
    }
    return count;
  };
  return atMost(right) - atMost(left - 1);
}
// O(n) time, O(1) space.
```
"Max in `[left, right]`" = "all elements ≤ right" minus "all elements ≤ left−1". Identical shape to `atMost(k) − atMost(k−1)` in LC 992 — recognize it as one idea. The `run = 0` reset on a too-large element is what makes it O(1) space instead of needing a map.

**31. Count Binary Substrings (LC 696)** — *not* a window. Run-length grouping.

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
Why no window works: valid substrings are constrained by *run structure*, and validity isn't monotone as you extend right — it flips on and off at every run boundary, which fails the Section 8 precondition. Two adjacent runs of lengths `a` and `b` contribute exactly `min(a, b)` valid substrings. This problem is in the set specifically to train pattern *rejection*: "contiguous substring, count them" is not automatically a window. Don't forget the final `Math.min` after the loop — the last run pair is never flushed inside it.

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
Two defences to have ready: the nested `while` is O(n) total because each swap permanently places a value, and the loop condition compares **values** (`nums[nums[i]-1] !== nums[i]`) so duplicates terminate — the index-based version loops forever on `[1,1]`. Also worth saying: the answer is always in `1..n+1`, which is why values outside that range can be ignored entirely. Test `[1,2,3]` → 4, `[7,8,9]` → 1, `[3,4,-1,1]` → 2.

### If you missed any

Mark it in `log.md` and re-solve after 3 days, then after 10. The specific ones worth extra attention because they carry the most transferable weight:

| Problem | Why it matters downstream |
|---|---|
| 8 (3Sum) | duplicate-skipping discipline reappears in every combinatorial problem in Phase 4 |
| 10 / 28 (min window) | Template C; if this is shaky, every "shortest valid" problem is shaky |
| 11 (Subarray Sum = K) | prefix + hash map is the standard escape when a window can't handle negatives |
| 31 (Count Binary Substrings) | pattern rejection — knowing when *not* to reach for a window |
| 32 (First Missing Positive) | the amortized argument for a nested `while`, which is the same argument as sliding window |

---
## 21. One-page cheat sheet

```
SIGNAL → PATTERN
  sorted + pair/triplet sum        two pointers from the ends
  palindrome / reverse in place    two pointers from the ends
  in place, return new length      write pointer
  contiguous + longest/shortest    sliding window (variable)
  window of size k                 sliding window (fixed)
  at most k distinct/zeros/swaps   variable window, shrink while violated
  exactly k                        atMost(k) − atMost(k−1)
  count subarrays where…           window, count += r − l + 1
  range sums / sum equals k        prefix sum (+ hash map to COUNT)
  divisible by k                   prefix sum mod k  → ((x%k)+k)%k
  equal #0s and #1s                map 0→−1, prefix revisits a value
  product except self              prefix × suffix
  max sum/profit/product subarray  Kadane (running value)
  values in 1..n, O(1) space       index-as-hash / cyclic sort
  merge two sorted, in place       fill from the BACK
  0s/1s/2s, one pass               Dutch national flag (3 pointers)
  unsorted + need indices          hash map (Phase 2), NOT sorting

THE FOUR SKELETONS

  write pointer                          two pointers, opposite ends
  let w = 0;                             let l = 0, r = n - 1;
  for (let r = 0; r < n; r++)             while (l < r) {
    if (keep(a[r])) a[w++] = a[r];          if (cond) l++; else r--;
  return w;                              }
  // O(n) / O(1)   invariant: w <= r     // O(n) / O(1)   needs SORTED

  window: LONGEST                        window: SHORTEST
  let l = 0, best = 0;                   let l = 0, best = Infinity;
  for (let r = 0; r < n; r++) {           for (let r = 0; r < n; r++) {
    add(a[r]);                              add(a[r]);
    while (INVALID) { rm(a[l]); l++; }      while (VALID) {
    best = max(best, r - l + 1);              best = min(best, r-l+1);
  }                                          rm(a[l]); l++;
                                           }
                                         }
                                         return best === Infinity ? 0 : best;

  LONGEST  → shrink while INVALID, record AFTER   → init 0
  SHORTEST → shrink while VALID,   record INSIDE  → init Infinity

FIXED WINDOW
  for (let i = 0; i < n; i++) {
    sum += a[i];
    if (i >= k)     sum -= a[i - k];       // element that LEAVES is a[i-k]
    if (i >= k - 1) best = max(best, sum); // only COMPLETE windows count
  }
  or: build first k, then  sum += a[i] - a[i-k]
  window start indices:  for (i = 0; i + k <= n; i++)

PREFIX SUM
  pre = new Array(n+1).fill(0);  pre[i+1] = pre[i] + a[i];
  sum(i..j) = pre[j+1] - pre[i]          // ALWAYS use the n+1 / pre[0]=0 form

  count subarrays == k:  seen = new Map([[0,1]]);
                         sum += x; count += seen.get(sum-k)||0;
                         seen.set(sum, (seen.get(sum)||0)+1);   // query BEFORE insert
  longest subarray == k: store FIRST index only  → if (!map.has(sum)) map.set(sum, i)
  shortest             : always overwrite
  2D:  pre[i+1][j+1] = m[i][j] + pre[i][j+1] + pre[i+1][j] - pre[i][j]
       rect = pre[r2+1][c2+1] - pre[r1][c2+1] - pre[r2+1][c1] + pre[r1][c1]
  range UPDATES:  diff[l] += v; diff[r+1] -= v;  then prefix-sum it

KADANE
  cur = best = a[0];                     // NOT 0 — all-negative arrays
  cur = max(a[i], cur + a[i]);           // "best ending here"
  best = max(best, cur);                 // "best anywhere"
  stock profit: best = 0 IS right (doing nothing is allowed) — read which one applies
  max PRODUCT: track max AND min, compute both candidates before assigning

WHY IT'S O(n)  (say this unprompted)
  right advances n times; left only moves forward, ≤ n times TOTAL
  → ≤ 2n pointer movements, not n per step
  inner counter RESETS  → multiply → O(n²)
  inner counter PERSISTS → add     → O(n)

WHEN THE WINDOW BREAKS
  negatives + "sum ≥ target"    → prefix sum + hash map / monotonic deque
  non-contiguous (subsequence)  → DP
  max in every window           → monotonic deque (Phase 6)
  precondition: validity must be MONOTONE as the window grows

COMPLEXITY
  write pointer / two pointers / window        O(n)   O(1)
  sort + two pointers                          O(n log n)
  3Sum  O(n²)      kSum  O(n^(k−1))            O(1) aux
  window + Map                                 O(n)   O(k)
  window + 26-array                            O(n)   O(1)
  prefix sum build/query                       O(n)/O(1)   O(n)
  prefix + hash map                            O(n)   O(n)
  all subarrays (brute)   O(n²), O(n³) if re-summing each

JS TRAPS FOR THIS PHASE
  Math.max(...arr)      RangeError on big arrays → loop or reduce
                        (fine on a FIXED 26-element array)
  sort()                mutates + string-compares → (a,b)=>a-b, and SAY it mutates
  %                     remainder, not modulo → ((x%k)+k)%k
  slice inside a loop    O(n) → O(n²).  Pass INDICES, slice once at the end
  arr[i] out of bounds   undefined → NaN silently.  NaN answer = OOB read
  new Array(n)           HOLEY → use .fill(0) or Array.from
  Array(n).fill([])      one shared reference → Array.from({length:n},()=>[])
  Object.keys().length   O(n) inside a loop → map.size is O(1)
  for…in on arrays       string keys + inherited props → never
  no int overflow        exact to 2^53; (lo+hi)>>1 breaks past 2^31
  strings immutable      no O(1)-space reversal; [...s] not s.split('') for emoji
  delete arr[i]          creates a hole → never

SIX OFF-BY-ONE CHECKS
  1  window length is r − l + 1
  2  bounds: windows i+k<=n · pairs i<n−1 · backward i>=0
  3  the element that leaves a fixed window is a[i−k]
  4  record AFTER the shrink (longest) / INSIDE it (shortest)
  5  init: 0 if empty answer legal · nums[0]/−Infinity if not · Infinity to minimize
  6  l<r for distinct pairs · l<=r when the middle must be processed
     strict > keeps the FIRST tie, >= keeps the LAST

DRY-RUN THESE FIVE, ALWAYS
  []   [5]   [2,2]/"aa"   all negatives   smallest case where answer ≠ whole array

INTERVIEW LINES
  open:   "brute force checks every pair, O(n²). It's sorted, so comparing the ends
           tells me which end can't be in the answer → O(n), O(1) space."
  window: "contiguous, and validity is monotone, so expand right / shrink left.
           Nested loop but O(n) — left never resets, ≤ n moves total."
  limits: "this needs all-positive values; with negatives I'd switch to prefix
           sums plus a hash map."
  close:  "O(n) time, O(1) auxiliary. Note it mutates the input — I'd copy first
           if the caller needs it preserved."
  stuck:  narrate the search over the signal table. Never go silent.
```

---

## Next

Phase 1 is done when you can:

- Write all four skeletons from memory, correctly, without re-deriving the boundaries.
- Decide longest-versus-shortest within 15 seconds of reading a window problem, and place the recording line correctly the first time.
- Give the O(n) amortized argument for sliding window unprompted, and the exchange argument for Container With Most Water.
- Say when a window *doesn't* apply — negatives, non-contiguous, max-per-window — and name the replacement.
- Solve 3Sum with all three duplicate skips, cold.

When that's true, come back and say **"Start Phase 2"** — hashing: `Map`/`Set` mechanics, frequency counters, grouping by computed key, and collapsing O(n²) brute force with O(1) lookups. It's the shortest phase and the highest hit rate per hour in the whole roadmap; roughly a third of easy/medium problems reduce to "put it in a Map."

Two things to carry forward from this phase into Phase 2:

- The **prefix + hash map** pairing (Section 10) is really a hashing pattern that happens to live here. Phase 2 will make the map side of it automatic.
- Every "use a `Set` instead of `includes`" instinct you built in Section 9's window states is the core Phase 2 skill, one phase early.
