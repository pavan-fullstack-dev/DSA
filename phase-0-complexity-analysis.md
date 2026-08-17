# Phase 0 — Complexity Analysis (JavaScript)

Complete reference. Everything you need for Days 1–2.

**Contents**

1. Why Big-O exists
2. The formal idea (and the three rules that follow from it)
3. Big-O vs Big-Θ vs Big-Ω
4. Best / average / worst case
5. The growth curves, with real numbers
6. Space complexity
7. A repeatable procedure for analyzing any code
8. Analyzing loops — every shape you'll meet
9. Analyzing recursion
10. Amortized analysis
11. The JavaScript cost table
12. `sort()` in depth
13. Objects vs `Map` vs `Set`
14. Strings in depth
15. The eleven traps that catch JS candidates
16. Pattern → complexity lookup
17. How to talk about it in an interview
18. 35 practice snippets
19. Worked answers
20. One-page cheat sheet

---

## 1. Why Big-O exists

You cannot compare two algorithms by timing them. A timing depends on your CPU, current thermal throttling, what else is running, whether V8 decided to JIT-compile the hot loop, and the specific input you picked. Run the same benchmark tomorrow and get a different number.

Big-O throws all of that away and asks one question:

> **As the input grows, how does the amount of work grow?**

That's it. Not "how fast is it" but "how does it scale." An O(n²) algorithm might beat an O(n log n) one on 50 elements. On 5 million it will lose by a margin measured in hours.

This is also why interviewers care. In a 40-minute interview they cannot benchmark your code. Complexity is the only objective, machine-independent way to judge whether your solution is good — so it becomes the scoring rubric.

**The mental model:** count the number of *elementary operations* your code performs as a function of the input size `n`, then describe the shape of that function while ignoring everything that doesn't affect the shape.

---

## 2. The formal idea

`f(n) = O(g(n))` means: there exist positive constants `c` and `n₀` such that for all `n ≥ n₀`,

```
f(n) ≤ c · g(n)
```

In plain terms: past some input size, `f` never grows faster than `g` scaled by a constant.

You will never write this in an interview. But three practical rules fall directly out of it, and those you use constantly.

### Rule 1 — Drop constant factors

```
O(2n)      → O(n)
O(500n)    → O(n)
O(n / 2)   → O(n)
O(3n log n) → O(n log n)
```

Why: constants are exactly what the `c` in the definition absorbs. Doubling the work doesn't change how it scales — it changes the machine you'd need, not the shape of the curve.

```js
// both O(n)
for (let i = 0; i < n; i++) doWork();
for (let i = 0; i < n; i += 2) doWork();  // half the iterations, same class
```

### Rule 2 — Drop lower-order terms

```
O(n² + n)         → O(n²)
O(n + log n)      → O(n)
O(n³ + 1000n²)    → O(n³)
O(2ⁿ + n⁵)        → O(2ⁿ)
```

Why: at n = 1,000,000, `n²` is a trillion and `n` is a million. The `n` is 0.0001% of the total. It's noise.

Keep the single fastest-growing term. Nothing else.

### Rule 3 — Different inputs get different variables

This one is genuinely misunderstood, and getting it right is a visible signal of care.

```js
function process(arrA, arrB) {
  for (const a of arrA) console.log(a);  // O(n)
  for (const b of arrB) console.log(b);  // O(m)
}
```

This is **O(n + m)**, not O(n). And not O(2n) — you cannot collapse two independent sizes into one variable. `arrB` could have 10 elements or 10 million; nothing about `arrA` tells you.

```js
function pairs(arrA, arrB) {
  for (const a of arrA)
    for (const b of arrB)
      console.log(a, b);
}
```

This is **O(n · m)**, not O(n²).

Only write O(n²) when both dimensions really are the same `n`.

---

## 3. Big-O vs Big-Θ vs Big-Ω

| Notation | Meaning | Analogy |
|---|---|---|
| **O(g)** | Upper bound — grows *no faster than* g | "at most" / ≤ |
| **Ω(g)** | Lower bound — grows *at least as fast as* g | "at least" / ≥ |
| **Θ(g)** | Tight bound — both upper and lower | "exactly this shape" / = |

Technically, an O(n) algorithm is *also* O(n²) and O(2ⁿ) — those are valid but useless upper bounds, like saying you're "under 500 years old."

In practice everyone says "O" and means Θ. When you say "this is O(n)" you're claiming it's tight.

**Where this matters:** if an interviewer asks "is bubble sort O(n²)?" the answer is yes. If they ask "is it Θ(n²)?" the answer is *no* — optimized bubble sort with an early-exit flag is Θ(n) on already-sorted input, so it's O(n²) but not Θ(n²). Knowing the distinction is a small, cheap credibility win. Don't volunteer it unprompted; it reads as pedantic.

---

## 4. Best / average / worst case

Different from the O/Ω/Θ distinction, and often confused with it. O/Ω/Θ describe *bounds on a function*. Best/average/worst describe *which input you're analyzing*.

You can have a Θ-bound on the worst case. They're orthogonal axes.

Linear search:

```js
function search(arr, target) {
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] === target) return i;
  }
  return -1;
}
```

| Case | Input | Complexity |
|---|---|---|
| Best | target is at index 0 | O(1) |
| Average | uniformly random position | O(n/2) = O(n) |
| Worst | target is last, or absent | O(n) |

**Default to worst case.** When someone says "this algorithm is O(n)" with no qualifier, they mean worst case. It's the only bound that's actually a guarantee — best case tells you nothing useful about whether your system will fall over.

### Where the cases genuinely diverge

| Algorithm | Best | Average | Worst |
|---|---|---|---|
| Linear search | O(1) | O(n) | O(n) |
| Binary search | O(1) | O(log n) | O(log n) |
| Quicksort | O(n log n) | O(n log n) | **O(n²)** |
| Mergesort | O(n log n) | O(n log n) | O(n log n) |
| Insertion sort | O(n) | O(n²) | O(n²) |
| Bubble sort (with early exit) | O(n) | O(n²) | O(n²) |
| Hash map lookup | O(1) | O(1) | **O(n)** |

Two rows deserve explanation:

**Quicksort's O(n²)** happens when the pivot is consistently the smallest or largest element — every partition splits into 0 and n−1 instead of n/2 and n/2. The recursion becomes n levels deep with O(n) work per level. Classic trigger: already-sorted input with a first-element pivot. Randomized or median-of-three pivot selection makes this vanishingly unlikely, which is why quicksort is used in practice despite the bad worst case.

**Hash map's O(n)** happens when every key collides into the same bucket, degenerating the structure into a linked list. With a decent hash function this doesn't occur naturally — but it *can* be induced deliberately (hash-flooding attacks), which is why V8 seeds its string hash randomly per process.

For interviews: state hash operations as O(1). If pressed on worst case, mention collisions and move on.

---

## 5. The growth curves, with real numbers

Ordered from best to worst:

```
O(1) < O(log n) < O(√n) < O(n) < O(n log n) < O(n²) < O(n³) < O(2ⁿ) < O(n!)
```

Operation counts (approximate, log base 2):

| n | O(1) | O(log n) | O(n) | O(n log n) | O(n²) | O(2ⁿ) | O(n!) |
|---|---|---|---|---|---|---|---|
| 10 | 1 | 3 | 10 | 33 | 100 | 1,024 | 3.6 million |
| 100 | 1 | 7 | 100 | 664 | 10,000 | 1.3 × 10³⁰ | 9 × 10¹⁵⁷ |
| 1,000 | 1 | 10 | 1,000 | 9,966 | 1,000,000 | ~10³⁰¹ | — |
| 10,000 | 1 | 13 | 10,000 | 132,877 | 100,000,000 | — | — |
| 1,000,000 | 1 | 20 | 1,000,000 | 19,931,569 | 10¹² | — | — |

Read the O(2ⁿ) column at n = 100: that number exceeds the count of atoms in the observable universe. Exponential is not "slow," it's *impossible*.

Read the O(log n) column at n = 1,000,000: twenty operations. Logarithmic is nearly free. This is why sorted data and balanced trees matter so much.

### What each complexity can actually handle

Assume ~10⁸ simple operations per second as a rough JS budget for a one-second response:

| Complexity | Feasible n |
|---|---|
| O(1) / O(log n) | anything |
| O(n) | ~100,000,000 |
| O(n log n) | ~5,000,000 |
| O(n²) | ~10,000 |
| O(n³) | ~500 |
| O(2ⁿ) | ~25 |
| O(n!) | ~11 |

Useful for a different reason too: **the constraint tells you the intended solution.** If a problem says n ≤ 20, it's hinting at exponential — bitmask or backtracking. If n ≤ 10⁵, you need O(n) or O(n log n), so brute force is out. If n ≤ 500, O(n³) DP is on the table.

### Where each shows up

**O(1) — constant.** Work independent of input size.
```js
arr[5]              // array index
map.get(key)        // hash lookup
obj.prop            // property access
arr.push(x)         // amortized
arr.pop()
stack.length
```

**O(log n) — logarithmic.** You eliminate a *fraction* of the remaining input each step. Almost always means halving.
```js
// binary search — halve the search space each iteration
// balanced BST operations
// heap insert / extract
// counting digits: while (n > 0) n = Math.floor(n / 10)
```
Recognition cue: a loop where the counter is multiplied or divided rather than incremented.

The base is irrelevant — log₂n, log₁₀n, and ln n differ only by a constant factor (`log_a n = log_b n / log_b a`), so all are O(log n).

**O(√n) — square root.** Rare but shows up in primality testing and factorization.
```js
for (let i = 2; i * i <= n; i++) { ... }
```

**O(n) — linear.** Touch each element a constant number of times.
```js
arr.map(f)
arr.filter(f)
arr.reduce(f)
arr.includes(x)
Math.max(...arr)
for (const x of arr) { ... }
```
Three separate passes over the array is still O(n) — that's Rule 1.

**O(n log n) — linearithmic.** The sorting barrier. Any comparison-based sort has an Ω(n log n) lower bound.
```js
arr.sort()
// mergesort, heapsort, quicksort (average)
// "sort first, then one pass" — extremely common pattern
```
Also arises from "do an O(log n) operation n times" — e.g. inserting n items into a heap or BST.

**O(n²) — quadratic.** Every pair.
```js
for (let i = 0; i < n; i++)
  for (let j = 0; j < n; j++)
    ...
```
Also: nested `includes`, bubble/selection/insertion sort, comparing all pairs. Most "optimize this" interview problems are asking you to turn O(n²) into O(n) with a hash map, or O(n log n) with sorting.

**O(2ⁿ) — exponential.** Two choices per element, explored fully.
```js
// naive Fibonacci
// generating all subsets (there are 2ⁿ of them)
// naive recursion with overlapping subproblems
```
Sometimes unavoidable — if the answer *is* all 2ⁿ subsets, you can't beat 2ⁿ. But usually it signals a missing memo.

**O(n!) — factorial.** All orderings.
```js
// generating all permutations (n! of them)
// brute-force travelling salesman
```

---

## 6. Space complexity

Same notation, applied to memory instead of time. Two variants, and the distinction matters:

- **Auxiliary space** — extra memory your algorithm allocates, excluding the input.
- **Total space** — auxiliary plus the input itself.

Interviewers almost always mean **auxiliary**. When you say "this is O(1) space," you mean you allocated a constant number of variables, even though the input array occupies O(n).

### What counts

```js
// O(1) auxiliary — a fixed number of scalars regardless of n
function sum(arr) {
  let total = 0;
  for (const x of arr) total += x;
  return total;
}

// O(n) auxiliary — output scales with input
function double(arr) {
  return arr.map(x => x * 2);
}

// O(n) auxiliary — the Set can hold up to n entries
function unique(arr) {
  return [...new Set(arr)];
}

// O(1) auxiliary — in-place, only pointers
function reverse(arr) {
  let l = 0, r = arr.length - 1;
  while (l < r) {
    [arr[l], arr[r]] = [arr[r], arr[l]];
    l++; r--;
  }
  return arr;
}
```

### The call stack counts as space

The single most-missed source of space complexity.

```js
function sumTo(n) {
  if (n === 0) return 0;
  return n + sumTo(n - 1);
}
```

Time O(n) — and space **O(n)**, because n frames pile up on the call stack before any of them return. Each frame holds its parameters, locals, and return address.

Compare:

```js
function sumTo(n) {
  let total = 0;
  for (let i = 1; i <= n; i++) total += i;
  return total;
}
```

Time O(n), space **O(1)**. Same time class, strictly better space. This is exactly the trade-off an interviewer probes when they ask "can you do this iteratively?"

**Recursion space = maximum depth of the call tree**, not the total number of calls.

For a balanced binary tree traversal: O(log n) space, because the tree is only log n deep even though you visit n nodes. For a degenerate (linked-list-shaped) tree: O(n).

### Two JS-specific facts about the stack

**The stack limit is real and low.** V8 allows roughly 10,000–15,000 frames before `RangeError: Maximum call stack size exceeded`. The exact number varies by frame size and engine version. Recursing over an array of 100,000 elements will crash — a genuine production concern, not a theoretical one.

**JavaScript has no tail-call optimization.** ES6 specified proper tail calls; V8 implemented and then removed them. Only JavaScriptCore (Safari) shipped it. So this still consumes O(n) stack in Node and Chrome despite being tail-recursive:

```js
function sumTo(n, acc = 0) {
  if (n === 0) return acc;
  return sumTo(n - 1, acc + n);   // tail call — still O(n) stack in V8
}
```

If you write a tail-recursive solution and claim O(1) space, you're wrong in JS. Say O(n), or convert to a loop.

### Time and space usually trade off

| Approach | Time | Space |
|---|---|---|
| Nested loop to find duplicates | O(n²) | O(1) |
| Sort, then scan adjacent | O(n log n) | O(1) or O(n) |
| Hash set | O(n) | O(n) |

There's rarely a universally correct choice. Saying *"I'll use a Set — that trades O(n) space for O(n) time instead of O(n²)"* demonstrates you understand there's a decision being made. That framing scores.

---

## 7. A repeatable procedure for analyzing any code

Run these steps in order. With practice it collapses into a glance, which is the Day 2 deliverable.

**Step 1 — Identify `n`.** What varies? Array length, string length, number of nodes, the numeric value of an input, number of digits. If there are two independent inputs, you need two variables.

**Step 2 — Find the deepest nesting.** Loops inside loops multiply. Sequential loops add.

**Step 3 — Check what each loop actually iterates.** `i++` → n iterations. `i *= 2` → log n. `i += 100` → still n (Rule 1). A bound that isn't n (like `i < 100`) → constant.

**Step 4 — Look inside the loop body for hidden work.** This is where most mistakes happen. `arr.includes()`, `str.slice()`, `[...arr]`, `.sort()`, `Object.keys()`, and `arr.indexOf()` are all O(n) or worse, and all look like single innocent operations. A loop body containing an O(n) call makes the loop O(n²).

**Step 5 — Handle recursion separately.** Count the branching factor and the depth (Section 9).

**Step 6 — Add, multiply, then simplify.** Sequential → add. Nested → multiply. Then drop constants and lower terms.

**Step 7 — Do space separately.** Count allocations that scale, plus maximum recursion depth.

---

## 8. Analyzing loops — every shape you'll meet

### Single loop → O(n)

```js
for (let i = 0; i < n; i++) console.log(i);
```

### Sequential loops → add, then simplify

```js
for (let i = 0; i < n; i++) console.log(i);   // O(n)
for (let j = 0; j < n; j++) console.log(j);   // O(n)
// O(n) + O(n) = O(2n) = O(n)
```

### Nested loops, both to n → O(n²)

```js
for (let i = 0; i < n; i++)
  for (let j = 0; j < n; j++)
    console.log(i, j);
```

### Nested with a constant inner bound → O(n)

```js
for (let i = 0; i < n; i++)
  for (let j = 0; j < 100; j++)   // fixed 100, not n
    console.log(i, j);
// 100n → O(n)
```

Nesting alone does not imply quadratic. Check what the inner loop is bounded by.

### Triangular nesting → still O(n²)

```js
for (let i = 0; i < n; i++)
  for (let j = i; j < n; j++)
    console.log(i, j);
```

Total iterations = n + (n−1) + (n−2) + … + 1 = n(n+1)/2 = (n² + n)/2.

Drop the constant ½ and the lower-order n → **O(n²)**.

Half the work of the full nested loop, same complexity class. A frequent trap: people see the `j = i` and guess O(n log n). It isn't.

### Multiplicative counter → O(log n)

```js
for (let i = 1; i < n; i *= 2) console.log(i);
```

i takes values 1, 2, 4, 8, … The loop runs until 2^k ≥ n, so k = log₂n.

Same for division:
```js
while (n > 1) n = Math.floor(n / 2);   // O(log n)
```

And for digit extraction — dividing by 10 gives log₁₀n steps, still O(log n):
```js
while (n > 0) { digit = n % 10; n = Math.floor(n / 10); }
```

### Outer linear, inner logarithmic → O(n log n)

```js
for (let i = 0; i < n; i++)
  for (let j = 1; j < n; j *= 2)
    console.log(i, j);
```

### Inner bound depends on the outer counter → O(log n)?  Careful.

```js
for (let i = 1; i < n; i *= 2)
  for (let j = 0; j < i; j++)
    console.log(i, j);
```

Inner runs i times, and i is 1, 2, 4, 8, …, up to n. Total = 1 + 2 + 4 + … + n = 2n − 1 → **O(n)**.

The geometric series is dominated by its last term. This is the same mathematics that makes `push` amortized O(1) (Section 10).

### Two pointers → O(n), even though it looks nested

```js
let left = 0, right = n - 1;
while (left < right) {
  if (someCondition) left++;
  else right--;
}
```

One iteration per pointer movement, and the pointers only ever move toward each other — at most n total movements. **O(n)**.

### Sliding window → O(n), even with a nested while

```js
let left = 0;
for (let right = 0; right < n; right++) {
  while (windowInvalid()) {
    left++;                 // inner loop!
  }
}
```

This looks O(n²) but is **O(n)**. `left` only ever increases, and it can increase at most n times *across the entire run* — not n times per outer iteration. Total pointer movements ≤ 2n.

This is an *amortized* argument (Section 10) and it's one of the most valuable things in this document. Interviewers specifically listen for whether you can explain why sliding window is linear. The wrong answer — "nested loops, so O(n²)" — is a common and costly mistake.

The test to apply: **does the inner loop's counter reset each outer iteration?** If it resets to 0, you're multiplying → O(n²). If it persists and only moves forward, you're adding → O(n).

```js
// resets → O(n²)
for (let i = 0; i < n; i++) {
  let j = 0;                     // reset
  while (j < n) j++;
}

// persists → O(n)
let j = 0;
for (let i = 0; i < n; i++) {
  while (j < n && cond(j)) j++;  // never reset
}
```

### Matrix loops

```js
// square matrix, n × n
for (let i = 0; i < n; i++)
  for (let j = 0; j < n; j++)
    grid[i][j];
// O(n²)  — but note: this is O(total cells), i.e. linear in input size
```

```js
// rectangular, rows × cols
for (let i = 0; i < rows; i++)
  for (let j = 0; j < cols; j++)
    grid[i][j];
// O(rows × cols)  — use two variables, per Rule 3
```

Nuance worth knowing: for an n × n grid the *input size* is n², so O(n²) time is actually optimal — you must read every cell. Some sources write this as O(m·n) or O(V) to make the point. Don't call it "quadratic and therefore bad."

---

## 9. Analyzing recursion

Two questions: **how many branches per call**, and **how deep does it go**.

### Linear recursion → O(n) time, O(n) space

```js
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
}
```

One call per level, n levels deep. Time O(n). Space O(n) — the call stack.

### Binary recursion, halving → O(log n)

```js
function binarySearch(arr, target, lo = 0, hi = arr.length - 1) {
  if (lo > hi) return -1;
  const mid = Math.floor((lo + hi) / 2);
  if (arr[mid] === target) return mid;
  return arr[mid] < target
    ? binarySearch(arr, target, mid + 1, hi)
    : binarySearch(arr, target, lo, mid - 1);
}
```

Two branches written, but only **one taken**. Depth log n. Time O(log n), space O(log n).

Counting written branches instead of taken branches is a frequent error.

### Binary recursion, both branches → O(2ⁿ)

```js
function fib(n) {
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2);
}
```

Two calls per level, depth n. The call tree has roughly 2ⁿ nodes. Time O(2ⁿ), space O(n) — space is *depth*, not node count, because the tree is explored depth-first and frames pop as you unwind.

(The tight bound is O(φⁿ) ≈ O(1.618ⁿ), since the tree isn't perfectly full. Say O(2ⁿ); mention φ only if asked to be precise.)

### Adding a memo → O(n)

```js
function fib(n, memo = {}) {
  if (n <= 1) return n;
  if (n in memo) return memo[n];
  return memo[n] = fib(n - 1, memo) + fib(n - 2, memo);
}
```

Each value of n is computed once. n distinct subproblems × O(1) work each → **O(n) time, O(n) space** (memo + stack).

The general rule for memoized recursion, and the one that makes DP complexity analysis mechanical:

> **time = (number of distinct states) × (work per state)**

### Divide and conquer → O(n log n)

```js
function mergeSort(arr) {
  if (arr.length <= 1) return arr;
  const mid = Math.floor(arr.length / 2);
  const left = mergeSort(arr.slice(0, mid));
  const right = mergeSort(arr.slice(mid));
  return merge(left, right);          // O(n)
}
```

Two recursive calls on half the input, plus O(n) merge work.

Think in levels: log n levels deep; each level does O(n) total merge work (the pieces at any level sum to n). Total **O(n log n)**.

Space is O(n) — the auxiliary arrays dominate the O(log n) stack.

### Recurrence relations you should recognize on sight

| Recurrence | Solution | Example |
|---|---|---|
| T(n) = T(n−1) + O(1) | O(n) | factorial, linear recursion |
| T(n) = T(n−1) + O(n) | O(n²) | naive quicksort worst case, selection sort |
| T(n) = T(n/2) + O(1) | O(log n) | binary search |
| T(n) = T(n/2) + O(n) | O(n) | quickselect average |
| T(n) = 2T(n/2) + O(1) | O(n) | tree traversal |
| T(n) = 2T(n/2) + O(n) | O(n log n) | mergesort |
| T(n) = 2T(n−1) + O(1) | O(2ⁿ) | subsets, naive fib, tower of hanoi |
| T(n) = nT(n−1) + O(1) | O(n!) | permutations |

Memorize this table. It converts most recursion analysis into pattern matching.

### Backtracking

```js
function subsets(nums) {
  const result = [];
  function backtrack(start, current) {
    result.push([...current]);          // O(n) copy!
    for (let i = start; i < nums.length; i++) {
      current.push(nums[i]);
      backtrack(i + 1, current);
      current.pop();
    }
  }
  backtrack(0, []);
  return result;
}
```

There are 2ⁿ subsets, and each is copied in O(n) → **O(n · 2ⁿ)** time.

Space: O(n) for the recursion depth and the `current` array, or O(n · 2ⁿ) if you count the output.

Permutations similarly: n! results × O(n) to copy each → **O(n · n!)**.

Don't forget the copy cost. Saying "O(2ⁿ)" for subsets is *almost* right and gets a follow-up question; "O(n · 2ⁿ) because each of the 2ⁿ subsets costs O(n) to copy into the result" ends the conversation.

### Tree recursion

```js
function maxDepth(root) {
  if (!root) return 0;
  return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}
```

Visits each of n nodes once → **O(n) time**.

Space = maximum recursion depth = tree height:
- balanced tree → O(log n)
- degenerate/skewed tree → O(n)

Always state both: *"O(n) time, O(h) space where h is the height — O(log n) balanced, O(n) worst case."* That single sentence covers what most candidates need two follow-up questions to reach.

---

## 10. Amortized analysis

Amortized complexity = **average cost per operation across a long sequence of operations**, where an occasional expensive operation is paid for by many cheap ones.

Critically, this is *not* the same as average-case. Average-case averages over random *inputs* and gives no guarantee for a specific input. Amortized averages over a *sequence of operations* and is a genuine guarantee: any sequence of n pushes costs O(n) total, always.

### Why `array.push` is O(1)

A JS array is backed by a contiguous block of memory with some *capacity* ≥ its length. Push writes into the next free slot: O(1).

When capacity runs out, the engine must:
1. allocate a bigger block (typically ~1.5–2× the old capacity)
2. copy all existing elements over — **O(n)**
3. write the new element

So a single push is usually O(1) but occasionally O(n). Why do we call it O(1)?

Take 16 pushes with doubling from capacity 1. Resizes happen at sizes 1, 2, 4, 8, 16, copying 1 + 2 + 4 + 8 + 16 = 31 elements total.

In general the total copy work across n pushes is:

```
1 + 2 + 4 + 8 + … + n  =  2n − 1  ≈  2n
```

A geometric series whose sum is a constant multiple of its largest term. So:

```
total cost for n pushes = O(n) writes + O(2n) copies = O(n)
cost per push           = O(n) / n = O(1) amortized
```

The expensive resizes are rare enough — and get exponentially rarer as the array grows — that they average out to a constant.

**The key insight is the growth factor.** Doubling gives amortized O(1). Growing by a *fixed amount* (say +1 each time) would give resizes at every push, total copy work of 1+2+3+…+n = O(n²), and amortized **O(n)** per push. The exponential growth factor is what makes it work.

### Other amortized-O(1) operations

- **`Map` / `Set` insertion** — hash tables rehash when the load factor is exceeded, an O(n) operation, amortized to O(1) by the same argument.
- **`arr.pop()`** — O(1); engines may shrink the backing store occasionally.
- **Sliding window pointer movement** — as in Section 8. Each element enters and leaves the window at most once, so the total inner-loop work across all outer iterations is ≤ 2n, giving O(1) amortized per outer step. Same reasoning, different setting.
- **Union-Find with path compression** — O(α(n)) amortized, effectively constant.

### How to say it

> "Push is amortized O(1). Individual pushes can be O(n) when the backing array resizes, but since capacity doubles, the total copy cost across n pushes is bounded by 2n — so it averages to constant time."

Three sentences, and it demonstrates you understand *why* rather than having memorized a table.

---

## 11. The JavaScript cost table

Complexities are for V8 (Chrome/Node) and hold for other major engines.

### Array — access & mutation

| Operation | Time | Notes |
|---|---|---|
| `arr[i]` | O(1) | direct index |
| `arr.length` | O(1) | stored, not computed |
| `arr.push(x)` | O(1) amortized | occasional resize |
| `arr.pop()` | O(1) | |
| `arr.shift()` | **O(n)** | every element reindexes down |
| `arr.unshift(x)` | **O(n)** | every element reindexes up |
| `arr.splice(i, d, ...items)` | **O(n)** | shifts everything after `i` |
| `arr.slice(a, b)` | O(k) | k = length of the slice; copies |
| `arr.concat(other)` | O(n + m) | new array |
| `[...arr]` | O(n) | copies |
| `arr.fill(x)` | O(n) | |
| `arr.reverse()` | O(n) | in place |
| `delete arr[i]` | O(1) | **creates a hole — avoid, deoptimizes the array** |

`shift` and `unshift` being O(n) is the most commonly missed entry in this table. Using `shift()` to dequeue in a BFS turns your O(V + E) traversal into O(V² + E). The fix is an index pointer:

```js
// O(n²) — shift is O(n) each time
while (queue.length) {
  const node = queue.shift();
  // ...
}

// O(n) — move a pointer instead of mutating
let head = 0;
while (head < queue.length) {
  const node = queue[head++];
  // ...
}
```

The pointer version trades O(n) memory (the array never shrinks) for the speedup. For large BFS use a real circular buffer or a linked list. But the index-pointer trick is what you write in an interview — it's three characters different and removes a whole complexity class.

(V8 does have optimizations that make `shift` cheaper on small arrays in some element kinds. Do not rely on this and do not cite it as a defense — state O(n).)

### Array — search & iteration

| Operation | Time | Notes |
|---|---|---|
| `arr.indexOf(x)` | O(n) | |
| `arr.lastIndexOf(x)` | O(n) | |
| `arr.includes(x)` | O(n) | linear scan — **not** a hash lookup |
| `arr.find(fn)` | O(n) | |
| `arr.findIndex(fn)` | O(n) | |
| `arr.some(fn)` / `arr.every(fn)` | O(n) | short-circuits, still O(n) worst case |
| `arr.filter(fn)` | O(n) | |
| `arr.map(fn)` | O(n) | assuming fn is O(1) |
| `arr.forEach(fn)` | O(n) | |
| `arr.reduce(fn)` | O(n) | assuming fn is O(1) |
| `arr.flat()` | O(n) | O(n·d) for depth d |
| `arr.flatMap(fn)` | O(n) | |
| `arr.join(sep)` | O(n) | |
| `arr.sort(cmp)` | O(n log n) | see Section 12 |
| `Math.max(...arr)` | O(n) | **also throws on large arrays** — see below |
| `Array.from(iterable)` | O(n) | |
| `Array.isArray(x)` | O(1) | |
| `new Array(n)` | O(1) | creates holes, doesn't initialize |
| `new Array(n).fill(0)` | O(n) | properly initialized |

`Math.max(...arr)` has a second problem beyond O(n): spreading passes each element as a separate function argument, and the engine's argument limit is around 65,000–125,000. Beyond that you get `RangeError: Maximum call stack size exceeded`. Use `arr.reduce((a, b) => Math.max(a, b), -Infinity)` for large arrays.

Note that `fn` being O(1) is an assumption, not a fact. `arr.map(x => arr.indexOf(x))` is O(n²).

### Object

| Operation | Time | Notes |
|---|---|---|
| `obj.key` / `obj[key]` | O(1) average | hash lookup |
| `obj.key = v` | O(1) average | |
| `key in obj` | O(1) | walks the prototype chain (bounded) |
| `obj.hasOwnProperty(key)` | O(1) | own properties only |
| `Object.hasOwn(obj, key)` | O(1) | modern preferred form |
| `delete obj.key` | O(1) | **may switch the object to dictionary mode — slow afterward** |
| `Object.keys(obj)` | O(n) | builds an array |
| `Object.values(obj)` | O(n) | |
| `Object.entries(obj)` | O(n) | |
| `Object.assign({}, obj)` | O(n) | shallow copy |
| `{ ...obj }` | O(n) | shallow copy |
| `Object.freeze(obj)` | O(n) | |
| `JSON.stringify(obj)` | O(n) | n = total nodes |
| `JSON.parse(str)` | O(n) | n = string length |
| `structuredClone(obj)` | O(n) | deep clone, handles cycles |

On `delete`: V8 represents objects with hidden classes (shapes) for fast property access. Deleting a property can force a transition to a slower dictionary-mode representation, and the object stays slow afterward. Prefer `obj.key = undefined`, or use a `Map` if you need real deletion.

### Map & Set

| Operation | Time |
|---|---|
| `map.get(k)` | O(1) average |
| `map.set(k, v)` | O(1) amortized |
| `map.has(k)` | O(1) average |
| `map.delete(k)` | O(1) average |
| `map.size` | O(1) |
| `map.clear()` | O(n) |
| iterating a Map/Set | O(n) |
| `set.add(x)` / `has` / `delete` | O(1) average |
| `[...set]` / `[...map]` | O(n) |
| `new Set(arr)` | O(n) |
| `new Map(entries)` | O(n) |

Worst case is O(n) under adversarial collisions. State O(1).

`WeakMap` / `WeakSet` are also O(1), hold keys weakly (garbage-collectable), and are not iterable. They're the correct tool for cycle detection in a deep-clone implementation, and for attaching metadata to DOM nodes without leaking.

### String

| Operation | Time | Notes |
|---|---|---|
| `str[i]` / `str.charAt(i)` | O(1) | |
| `str.length` | O(1) | |
| `str.charCodeAt(i)` / `codePointAt(i)` | O(1) | |
| `str + other` | O(n + m) | see Section 14 for the V8 nuance |
| `str.slice(a, b)` / `substring` | O(k) | copies k chars |
| `str.indexOf(sub)` | O(n · m) worst | typically much better |
| `str.includes(sub)` | O(n · m) worst | |
| `str.split(sep)` | O(n) | |
| `str.toUpperCase()` / `toLowerCase()` | O(n) | |
| `str.trim()` | O(n) | |
| `str.repeat(k)` | O(n · k) | |
| `str.replace(str, x)` | O(n) | first match only |
| `str.replaceAll(str, x)` | O(n) | |
| `str.padStart` / `padEnd` | O(n) | |
| `[...str]` / `str.split('')` | O(n) | |
| `str.match(regex)` | O(n) to **O(2ⁿ)** | catastrophic backtracking is real |
| template literal | O(n) | |
| `localeCompare` | O(n) | much slower constant than `<` |

On regex: nested quantifiers like `/(a+)+b/` can backtrack exponentially. This is the ReDoS vulnerability class, and it's a legitimate production concern in input-validation code. Worth mentioning if a problem hands you user-supplied patterns.

### Numbers & math

| Operation | Time |
|---|---|
| arithmetic `+ - * / %` | O(1) |
| `Math.floor` / `ceil` / `round` / `abs` | O(1) |
| `Math.sqrt` / `pow` / `log` | O(1) |
| bitwise `& \| ^ ~ << >>` | O(1) |
| `n.toString()` | O(log n) — digits |
| `parseInt(str)` | O(n) |
| `Number.isInteger(n)` | O(1) |
| BigInt arithmetic | O(d) to O(d²) — d = digits |

Careful with the distinction between **input value** and **input size**. Counting the digits of a number is O(log n) in the *value* of n, which is O(d) in the number of digits. Both are correct; be explicit about which you mean.

### Typed arrays & other structures

| Operation | Time |
|---|---|
| `Int32Array` / `Float64Array` index access | O(1), better constants than `Array` |
| `arr.sort()` on a TypedArray | O(n log n), numeric by default |
| `Date.now()` | O(1) |
| `structuredClone` | O(n) |

---

## 12. `sort()` in depth

Four things to know, all of which appear in interviews.

### 1. It mutates

```js
const arr = [3, 1, 2];
const sorted = arr.sort();
console.log(arr);            // [1, 2, 3] — original changed!
console.log(sorted === arr); // true — same reference
```

If the input must be preserved: `const sorted = [...arr].sort()` or `arr.toSorted()` (ES2023 — check your Node/browser target before relying on it).

Mutating an input array you were handed is a real bug in React code, where it defeats reference-equality change detection. Interviewers who've reviewed a lot of frontend code notice this.

### 2. The default comparator converts to strings

```js
[10, 9, 100, 1].sort();
// [1, 10, 100, 9]   ← not a bug, documented behavior
```

Elements are converted to strings and compared by UTF-16 code unit. `"10" < "9"` because `"1"` < `"9"`.

```js
[10, 9, 100, 1].sort((a, b) => a - b);   // [1, 9, 10, 100] ✅
```

Comparator contract: return negative if `a` comes first, positive if `b` comes first, zero if equivalent.

```js
arr.sort((a, b) => a - b);                          // numbers ascending
arr.sort((a, b) => b - a);                          // numbers descending
arr.sort((a, b) => a.localeCompare(b));             // strings, locale-aware
arr.sort((a, b) => a.age - b.age);                  // objects by field
arr.sort((a, b) => a.age - b.age || a.name.localeCompare(b.name));  // multi-key
```

Never write `arr.sort((a, b) => a > b)` — that returns a boolean, coerces to 0 or 1, never negative, and produces subtly wrong results.

### 3. Complexity is O(n log n) — with the comparator's cost multiplied in

V8 has used **TimSort** since v7.0 (Chrome 70, 2018) — a hybrid of mergesort and insertion sort:

- Worst case O(n log n)
- **Best case O(n)** on already-sorted or nearly-sorted input
- **Stable** — equal elements keep their relative order (guaranteed by the spec since ES2019)
- O(n) auxiliary space

The real cost is O(n log n × cost of comparator). So:

```js
arr.sort((a, b) => a - b);                              // O(n log n)
arr.sort((a, b) => a.name.localeCompare(b.name));       // O(n log n × k), k = string length
arr.sort((a, b) => expensiveScore(a) - expensiveScore(b)); // recomputes ~2n log n times!
```

For expensive keys, precompute once — the Schwartzian transform:

```js
const sorted = arr
  .map(x => ({ key: expensiveScore(x), value: x }))   // n calls, not n log n
  .sort((a, b) => a.key - b.key)
  .map(x => x.value);
```

### 4. The lower bound

Any comparison-based sort requires Ω(n log n) comparisons — there are n! possible orderings and each comparison yields one bit, so you need at least log₂(n!) = Θ(n log n) comparisons.

You can beat it by **not comparing**: counting sort, radix sort, and bucket sort achieve O(n + k) or O(d·n) by exploiting structure in the keys. Relevant when a problem says "the values are in the range 0 to 100" — that's a hint toward counting sort.

---

## 13. Objects vs `Map` vs `Set`

All O(1) for the core operations. Choose on semantics, not speed.

| | Object | Map | Set |
|---|---|---|---|
| Key types | string, symbol (numbers coerce to strings) | **any** — objects, functions, NaN | values, any type |
| Insertion order | mostly preserved, but integer-like keys sort first | **guaranteed** | guaranteed |
| Size | `Object.keys(o).length` — **O(n)** | `map.size` — **O(1)** | `set.size` — O(1) |
| Iteration | needs `Object.keys/entries` | directly iterable | directly iterable |
| Prototype keys | inherits `toString`, `constructor`, etc. | none | none |
| Frequent add/delete | can deoptimize | optimized for it | optimized for it |
| JSON serializable | yes | no (needs conversion) | no |

### The integer-key ordering surprise

```js
const obj = { b: 1, a: 2, 2: 3, 1: 4 };
Object.keys(obj);   // ["1", "2", "b", "a"]  ← integer keys first, ascending
```

Integer-like keys are enumerated in ascending numeric order before string keys, regardless of insertion order. If order matters and your keys look numeric, use a `Map`.

### Key coercion

```js
const obj = {};
obj[1] = "number";
obj["1"] = "string";
console.log(obj);        // { "1": "string" }  ← collided!

const map = new Map();
map.set(1, "number");
map.set("1", "string");
console.log(map.size);   // 2  ← distinct keys
```

Also: `obj[{a:1}] = "x"` stringifies the key to `"[object Object]"`, so all objects collide into one key. `Map` handles object keys correctly by identity.

### The prototype trap

```js
const obj = {};
console.log("toString" in obj);       // true! inherited
console.log(obj.constructor);         // ƒ Object()

// so a naive frequency counter breaks on the word "constructor":
const counts = {};
counts["constructor"] = (counts["constructor"] || 0) + 1;  // works, but
if (counts["toString"]) { /* truthy even though never set! */ }
```

Fixes: `Object.create(null)` for a prototype-less object, `Object.hasOwn()` instead of truthiness checks, or just use a `Map`.

### Practical guidance

Use a **Map** when: keys aren't strings, you add/delete frequently, order matters, you need `.size`, or keys could collide with prototype properties.

Use an **object** when: fixed known string keys, you need JSON serialization, or you're writing a small frequency counter with safe keys.

Use a **Set** when: you only care about presence, not an associated value.

In DSA problems, `Map` and `Set` are almost always the right call. A plain object is fine for a character-frequency counter (`a`–`z` keys are safe), and marginally faster there.

---

## 14. Strings in depth

### Immutability

```js
let s = "hello";
s[0] = "H";
console.log(s);   // "hello" — silently unchanged
```

Every "modification" creates a new string. This is the root of the concatenation problem.

### The concatenation trap — with the honest V8 caveat

The textbook analysis:

```js
let result = "";
for (let i = 0; i < n; i++) {
  result += "x";        // allegedly copies i characters each time
}
// 1 + 2 + 3 + … + n = O(n²)
```

**The nuance:** V8 does not actually do this. It builds a **rope** (a "cons string") — a lazy tree of string fragments — and only flattens into a contiguous buffer when the string is actually read (indexed, compared, passed to a C++ API). So in practice `+=` in a loop is close to O(n) in V8, and often *faster* than array-join because it avoids the intermediate array.

**What to say in an interview:** lead with the algorithmic answer, then show you know the engine detail.

> "Strings are immutable, so the naive model says repeated concatenation is O(n²) — each `+=` copies the accumulated string. In practice V8 uses rope structures and defers flattening, so it's closer to linear. But I'd still use an array and `join` because it's O(n) by construction and doesn't depend on an engine optimization."

That answer is strictly better than either "it's O(n²)" (incomplete) or "V8 optimizes it so it's fine" (sounds like you don't know the underlying model). The safe, portable pattern:

```js
const parts = [];
for (let i = 0; i < n; i++) parts.push("x");
const result = parts.join("");    // O(n)
```

### Slicing is not free

```js
// O(n²) — every recursive call copies the remaining string
function reverse(str) {
  if (str.length <= 1) return str;
  return reverse(str.slice(1)) + str[0];
}

// O(n) — index arithmetic, no copies
function reverse(str) {
  let out = "";
  for (let i = str.length - 1; i >= 0; i--) out += str[i];
  return out;
}
```

Prefer passing indices over slicing substrings in recursion. Applies to arrays too: `arr.slice(1)` inside a recursive call is a silent O(n) per level.

### Unicode: length lies

```js
"café".length;      // 4 ✓
"👨‍👩‍👧".length;       // 8  — one visible glyph, eight UTF-16 code units
"é".length;         // 1 or 2, depending on NFC vs NFD normalization
```

JS strings are sequences of UTF-16 code units. Characters outside the Basic Multilingual Plane (emoji, many CJK extensions) use surrogate pairs and count as 2.

```js
[..."👨‍👩‍👧"].length;                          // 5 — code points, still not 1
Array.from("👨‍👩‍👧").length;                    // 5
[...new Intl.Segmenter().segment("👨‍👩‍👧")].length; // 1 — grapheme clusters ✓
```

For a palindrome or reversal problem with emoji, naive index-based iteration will split surrogate pairs and corrupt the output. Rarely tested, but flagging it — *"I'm assuming ASCII input; with Unicode I'd need to iterate code points or grapheme clusters"* — is a strong senior-level signal that costs one sentence.

### Comparison

```js
"a" < "b"                    // O(1)–O(n), code-unit comparison, fast
"a".localeCompare("b")       // O(n) with a much larger constant — but correct for i18n
```

In a sort comparator over 100,000 strings, `localeCompare` can be an order of magnitude slower. Use `<` for pure algorithmic work, `localeCompare` when the ordering is user-facing.

---

## 15. The eleven traps that catch JS candidates

### Trap 1 — `includes` inside a loop

```js
// O(n²)
function findDuplicates(arr) {
  const dupes = [];
  for (const x of arr) {
    if (dupes.includes(x)) continue;      // O(n) inside an O(n) loop
    if (arr.indexOf(x) !== arr.lastIndexOf(x)) dupes.push(x);  // two more O(n) scans
  }
  return dupes;
}

// O(n)
function findDuplicates(arr) {
  const seen = new Set(), dupes = new Set();
  for (const x of arr) {
    if (seen.has(x)) dupes.add(x);
    else seen.add(x);
  }
  return [...dupes];
}
```

The single highest-frequency mistake. Any `includes` / `indexOf` / `find` inside a loop should trigger the thought *"can this be a Set?"*

### Trap 2 — spread inside `reduce`

```js
// O(n²) — extremely common in React/Redux code
const byId = items.reduce((acc, item) => ({ ...acc, [item.id]: item }), {});
```

Each iteration copies the whole accumulator: 1 + 2 + 3 + … + n = O(n²). At 10,000 items this is 50 million property copies.

```js
// O(n) — mutate the accumulator
const byId = items.reduce((acc, item) => {
  acc[item.id] = item;
  return acc;
}, {});

// O(n) — or clearer
const byId = Object.fromEntries(items.map(i => [i.id, i]));
```

Same trap with arrays: `acc.concat(x)` and `[...acc, x]` inside a reduce are both O(n²). Use `acc.push(x); return acc;`.

Worth calling out explicitly if you're asked to review code — spotting this in a PR is a concrete, demonstrable skill.

### Trap 3 — `shift()` as dequeue

Covered in Section 11. Turns BFS from O(V + E) into O(V² + E). Use an index pointer.

### Trap 4 — `splice` in a loop

```js
// O(n²)
for (let i = arr.length - 1; i >= 0; i--) {
  if (shouldRemove(arr[i])) arr.splice(i, 1);   // O(n) each
}

// O(n)
const filtered = arr.filter(x => !shouldRemove(x));
```

### Trap 5 — `sort` inside a loop

```js
// O(n² log n)
for (let i = 0; i < n; i++) {
  const sorted = [...arr].sort();   // O(n log n) per iteration
  use(sorted);
}
```
Sort once outside the loop.

### Trap 6 — recomputing `Object.keys` in a loop

```js
// O(n²)
for (let i = 0; i < n; i++) {
  if (Object.keys(obj).length > 5) { ... }   // O(n) each iteration
}
```
Hoist it, or use a `Map` and its O(1) `.size`.

### Trap 7 — nested `filter`/`map` over two arrays

```js
// O(n · m) — often accidental
const enriched = users.map(u => ({
  ...u,
  orders: orders.filter(o => o.userId === u.id)    // full scan per user
}));

// O(n + m) — index first
const byUser = new Map();
for (const o of orders) {
  if (!byUser.has(o.userId)) byUser.set(o.userId, []);
  byUser.get(o.userId).push(o);
}
const enriched = users.map(u => ({ ...u, orders: byUser.get(u.id) ?? [] }));
```

This exact shape appears constantly in real frontend data-joining code. It's a strong answer to "tell me about a performance problem you fixed."

### Trap 8 — hidden O(n) copies in recursion

```js
// O(n²) — slice copies at every level
function sum(arr) {
  if (!arr.length) return 0;
  return arr[0] + sum(arr.slice(1));
}

// O(n) — pass an index
function sum(arr, i = 0) {
  if (i >= arr.length) return 0;
  return arr[i] + sum(arr, i + 1);
}
```

### Trap 9 — assuming the callback is O(1)

```js
arr.map(x => arr.indexOf(x));            // O(n²)
arr.filter(x => other.includes(x));      // O(n · m)
arr.some(x => heavyCheck(x));            // depends entirely on heavyCheck
```

`map` is O(n) *times the cost of the callback*. Always look inside.

### Trap 10 — string building without an array

Section 14. Use `parts.push()` + `join('')` for portability and a guaranteed bound.

### Trap 11 — `delete` and array holes

```js
const arr = [1, 2, 3];
delete arr[1];
console.log(arr);          // [1, empty, 3]
console.log(arr.length);   // 3 — length unchanged
```

Creates a sparse array. V8 downgrades it from a fast packed representation (`PACKED_SMI_ELEMENTS`) to a slow holey one (`HOLEY_ELEMENTS`), and every subsequent access pays a prototype-chain check. Same for `new Array(5)` without `.fill()`, and for assigning past the end (`arr[1000] = 1` on a 3-element array).

Use `splice` to actually remove, or `filter` to produce a new dense array.

---

## 16. Pattern → complexity lookup

Memorize this. It's most of the "answer in 10 seconds" skill.

| Code shape | Complexity |
|---|---|
| Single pass over the input | O(n) |
| Two sequential passes | O(n) |
| Nested loop, both to n | O(n²) |
| Triangular nested loop (`j = i`) | O(n²) |
| Nested loop, inner bound constant | O(n) |
| Counter multiplied or divided | O(log n) |
| Linear outer × logarithmic inner | O(n log n) |
| Sort, then single pass | O(n log n) |
| Two pointers converging | O(n) |
| Sliding window (non-resetting inner) | O(n) |
| Hash map single pass | O(n) time, O(n) space |
| Binary search | O(log n) |
| Binary search inside a loop | O(n log n) |
| Traverse every node of a tree/graph once | O(n) or O(V + E) |
| BFS/DFS on a grid | O(rows × cols) |
| Heap operations n times | O(n log n) |
| Build a heap from an array | O(n) — not O(n log n) |
| Recursion, one branch, depth n | O(n) |
| Recursion, two branches, depth n | O(2ⁿ) |
| Recursion, two branches, halving | O(n) or O(n log n) |
| Memoized recursion | O(states × work per state) |
| Generate all subsets | O(n · 2ⁿ) |
| Generate all permutations | O(n · n!) |
| Backtracking with pruning | exponential, better in practice |

Two entries deserve a note:

**Build a heap from an array is O(n), not O(n log n).** Sifting down from the middle to the front — nodes near the leaves (most of them) sift a short distance, and the sum works out to O(n). Inserting n items one at a time *is* O(n log n). A nice detail to have ready for Phase 8.

**Binary search inside a loop.** Very common composite: `for (const x of arr) binarySearch(sorted, x)` → O(n log n). Recognize the composition rather than analyzing from scratch.

---

## 17. How to talk about it in an interview

### Before you write code

> "Brute force here is O(n²) — check every pair. I think we can get to O(n) with a hash map by trading O(n) space. Want me to go straight to the optimal version, or start with brute force and improve it?"

This does several things at once: shows you thought before typing, names the trade-off, and hands the interviewer control of the direction. It's the strongest possible opening.

### After you write code

> "Time is O(n log n) — dominated by the sort; the scan after it is O(n). Space is O(n) for the copy, or O(1) if I'm allowed to sort in place. The worst case is when all elements are distinct, which doesn't change the bound here."

### Structure to hit every time

1. Name the time complexity and **say what dominates**
2. Name the space complexity and **say what's consuming it**
3. Note whether it's worst / average / amortized if that's non-obvious
4. State the trade-off you chose and what the alternative was

### Phrases that land well

- "The sort dominates, so the linear scan after it doesn't change the bound."
- "That's amortized O(1) — resizes are O(n) but exponentially rare."
- "O(h) space where h is the tree height, so O(log n) balanced and O(n) skewed."
- "It's O(n + m), not O(n²) — the two inputs are independent."
- "I'm trading O(n) space for O(n) time instead of O(n²)."
- "The inner while doesn't reset, so total pointer movement is bounded by 2n — it's O(n), not O(n²)."
- "I'd avoid `shift` here; it's O(n), so I'll use an index pointer instead."

### Mistakes to avoid

- Guessing a complexity without walking the loops
- Ignoring the cost of built-in methods
- Forgetting the call stack in space complexity
- Calling sliding window O(n²)
- Claiming O(1) space for tail recursion in JS
- Saying "O(2n)" or "O(n/2)" instead of simplifying
- Collapsing two independent inputs into one variable
- Not mentioning space at all until asked

That last one matters more than people expect. Volunteering the space complexity unprompted is a small, consistent differentiator — most candidates wait to be asked.

---

## 18. Practice — 35 snippets

State time **and** space for each. Answers in Section 19. Cover them and work through it; then re-do the ones you missed after three days.

```js
// 1
function q1(n) {
  let sum = 0;
  for (let i = 0; i < n; i++) sum += i;
  return sum;
}

// 2
function q2(n) {
  let sum = 0;
  for (let i = 0; i < n; i++)
    for (let j = 0; j < n; j++)
      sum += i * j;
  return sum;
}

// 3
function q3(n) {
  for (let i = 0; i < n; i++) console.log(i);
  for (let j = 0; j < n; j++) console.log(j);
  for (let k = 0; k < n; k++) console.log(k);
}

// 4
function q4(n) {
  for (let i = 1; i < n; i *= 2) console.log(i);
}

// 5
function q5(n) {
  for (let i = 0; i < n; i++)
    for (let j = 0; j < 50; j++)
      console.log(i, j);
}

// 6
function q6(arr) {
  for (let i = 0; i < arr.length; i++)
    for (let j = i + 1; j < arr.length; j++)
      if (arr[i] === arr[j]) return true;
  return false;
}

// 7
function q7(arr) {
  return new Set(arr).size !== arr.length;
}

// 8
function q8(n) {
  for (let i = 0; i < n; i++)
    for (let j = 1; j < n; j *= 2)
      console.log(i, j);
}

// 9
function q9(arr) {
  return arr.map(x => x * 2).filter(x => x > 10).reduce((a, b) => a + b, 0);
}

// 10
function q10(arr) {
  return arr.map(x => arr.indexOf(x));
}

// 11
function q11(a, b) {
  for (const x of a)
    for (const y of b)
      console.log(x, y);
}

// 12
function q12(str) {
  let out = "";
  for (let i = str.length - 1; i >= 0; i--) out += str[i];
  return out;
}

// 13
function q13(arr) {
  const sorted = [...arr].sort((a, b) => a - b);
  for (let i = 1; i < sorted.length; i++)
    if (sorted[i] === sorted[i - 1]) return sorted[i];
  return null;
}

// 14
function q14(n) {
  if (n <= 1) return n;
  return q14(n - 1) + q14(n - 2);
}

// 15
function q15(n, memo = {}) {
  if (n <= 1) return n;
  if (n in memo) return memo[n];
  return memo[n] = q15(n - 1, memo) + q15(n - 2, memo);
}

// 16
function q16(arr, target) {
  let lo = 0, hi = arr.length - 1;
  while (lo <= hi) {
    const mid = (lo + hi) >> 1;
    if (arr[mid] === target) return mid;
    if (arr[mid] < target) lo = mid + 1;
    else hi = mid - 1;
  }
  return -1;
}

// 17
function q17(arr) {
  const result = [];
  for (const x of arr) result.unshift(x);
  return result;
}

// 18
function q18(items) {
  return items.reduce((acc, item) => ({ ...acc, [item.id]: item }), {});
}

// 19
function q19(root) {
  if (!root) return 0;
  return 1 + Math.max(q19(root.left), q19(root.right));
}

// 20
function q20(grid) {
  for (let i = 0; i < grid.length; i++)
    for (let j = 0; j < grid[0].length; j++)
      console.log(grid[i][j]);
}

// 21
function q21(n) {
  let count = 0;
  for (let i = 0; i < n; i++)
    for (let j = i; j < n; j++)
      count++;
  return count;
}

// 22
function q22(nums) {
  const result = [];
  function bt(start, cur) {
    result.push([...cur]);
    for (let i = start; i < nums.length; i++) {
      cur.push(nums[i]);
      bt(i + 1, cur);
      cur.pop();
    }
  }
  bt(0, []);
  return result;
}

// 23
function q23(s) {
  let left = 0, best = 0;
  const seen = new Set();
  for (let right = 0; right < s.length; right++) {
    while (seen.has(s[right])) {
      seen.delete(s[left]);
      left++;
    }
    seen.add(s[right]);
    best = Math.max(best, right - left + 1);
  }
  return best;
}

// 24
function q24(arr) {
  if (arr.length <= 1) return arr;
  const mid = arr.length >> 1;
  const l = q24(arr.slice(0, mid));
  const r = q24(arr.slice(mid));
  const out = [];
  let i = 0, j = 0;
  while (i < l.length && j < r.length) out.push(l[i] <= r[j] ? l[i++] : r[j++]);
  return out.concat(l.slice(i)).concat(r.slice(j));
}

// 25
function q25(n) {
  let i = n;
  while (i > 1) i = Math.floor(i / 2);
}

// 26
function q26(str) {
  if (str.length <= 1) return str;
  return q26(str.slice(1)) + str[0];
}

// 27
function q27(users, orders) {
  return users.map(u => ({
    ...u,
    orders: orders.filter(o => o.userId === u.id)
  }));
}

// 28
function q28(n) {
  for (let i = 0; i < n; i++) {
    let j = 0;
    while (j < n) j++;
  }
}

// 29
function q29(n) {
  let j = 0;
  for (let i = 0; i < n; i++) {
    while (j < n && j < i * 2) j++;
  }
}

// 30
function q30(arr) {
  const queue = [...arr];
  const out = [];
  while (queue.length) out.push(queue.shift());
  return out;
}

// 31
function q31(n) {
  for (let i = 1; i <= n; i++)
    for (let j = 1; j <= n; j += i)
      console.log(i, j);
}

// 32
function q32(arr) {
  const counts = new Map();
  for (const x of arr) counts.set(x, (counts.get(x) ?? 0) + 1);
  return [...counts.entries()].sort((a, b) => b[1] - a[1]).slice(0, 10);
}

// 33
function q33(n) {
  for (let i = 2; i * i <= n; i++)
    if (n % i === 0) return false;
  return n > 1;
}

// 34
function q34(nums) {
  const out = [];
  function perm(cur, remaining) {
    if (!remaining.length) { out.push([...cur]); return; }
    for (let i = 0; i < remaining.length; i++) {
      cur.push(remaining[i]);
      perm(cur, [...remaining.slice(0, i), ...remaining.slice(i + 1)]);
      cur.pop();
    }
  }
  perm([], nums);
  return out;
}

// 35
function q35(matrix) {
  const n = matrix.length;
  for (let i = 0; i < n; i++)
    for (let j = i + 1; j < n; j++)
      [matrix[i][j], matrix[j][i]] = [matrix[j][i], matrix[i][j]];
  for (const row of matrix) row.reverse();
  return matrix;
}
```

---

## 19. Answers

**1.** O(n) time, O(1) space. Single loop, fixed variables.

**2.** O(n²) time, O(1) space. Both loops bound by n.

**3.** O(n) time, O(1) space. Three sequential passes = O(3n) = O(n). Rule 1.

**4.** O(log n) time, O(1) space. `i *= 2` reaches n in log₂n steps.

**5.** O(n) time, O(1) space. Inner bound is 50, a constant → O(50n) = O(n).

**6.** O(n²) time, O(1) space. Triangular nesting: n(n−1)/2 comparisons.

**7.** O(n) time, O(n) space. `new Set(arr)` is one pass; the Set holds up to n. Same job as #6 with a space-for-time trade.

**8.** O(n log n) time, O(1) space. Linear outer × logarithmic inner.

**9.** O(n) time, O(n) space. Three sequential O(n) passes; `map` and `filter` each allocate a new array.

**10.** **O(n²)** time, O(n) space. `indexOf` is O(n) and runs n times. The classic "callback isn't O(1)" trap.

**11.** O(n · m) time, O(1) space. Independent inputs — not O(n²).

**12.** O(n) time, O(n) space. Naive model says O(n²) for the concatenation; V8's ropes make it effectively linear. State it as O(n) with the caveat, or use `join`.

**13.** O(n log n) time, O(n) space. Sort dominates; the scan is O(n). The spread copy is O(n) space. Sorting in place would give O(1) auxiliary but mutate the input.

**14.** O(2ⁿ) time, O(n) space. Tight bound O(φⁿ) ≈ O(1.618ⁿ). Space is depth, not node count.

**15.** O(n) time, O(n) space. n distinct states × O(1) each. Space = memo + call stack.

**16.** O(log n) time, O(1) space. Iterative, so no stack growth. The recursive version would be O(log n) space.

**17.** **O(n²)** time, O(n) space. `unshift` is O(n) and runs n times. `push` then `reverse` would be O(n).

**18.** **O(n²)** time, O(n) space. Spread inside reduce copies the growing accumulator. Trap 2.

**19.** O(n) time, O(h) space — h = height. O(log n) balanced, O(n) skewed. Say both.

**20.** O(rows × cols) time, O(1) space. Note `grid[0]` breaks on an empty grid — an edge case worth mentioning aloud.

**21.** O(n²) time, O(1) space. Triangular again. The result is n(n+1)/2, which is itself the giveaway.

**22.** O(n · 2ⁿ) time. 2ⁿ subsets, each O(n) to copy. Space O(n) auxiliary (stack + `cur`), or O(n · 2ⁿ) counting output.

**23.** **O(n)** time, O(min(n, k)) space where k is the alphabet size. The inner `while` doesn't reset — `left` advances at most n times total across the whole run. Sliding window. If you said O(n²), reread Section 8.

**24.** O(n log n) time, O(n) space. Mergesort. `slice` and `concat` allocate; that's the O(n) space.

**25.** O(log n) time, O(1) space. Repeated halving.

**26.** **O(n²)** time, O(n) space. `slice(1)` copies at every one of n recursion levels. Trap 8. Contrast with #12, which is O(n).

**27.** O(n · m) time, O(n · m) space. Full scan of orders per user. Trap 7 — fix by indexing orders into a Map first, giving O(n + m).

**28.** O(n²) time, O(1) space. `j` **resets** to 0 each outer iteration.

**29.** O(n) time, O(1) space. `j` is declared outside and never resets — total movement bounded by n. Compare directly with #28; the only difference is where `j` lives, and it's a whole complexity class.

**30.** **O(n²)** time, O(n) space. `shift` is O(n), called n times. Trap 3 — this is exactly the BFS bug.

**31.** O(n log n) time, O(1) space. Inner loop runs ~n/i times; total = n(1/1 + 1/2 + 1/3 + … + 1/n) = n · H(n) ≈ n ln n. The harmonic series — the one genuinely tricky snippet here.

**32.** O(n + k log k) time where k = distinct values, so O(n log n) worst case. Space O(k). Note that a heap would give O(n log 10) = O(n) for top-10 — a good thing to volunteer.

**33.** O(√n) time, O(1) space. Note this is in the *value* of n; in the number of bits it's exponential. Primality trial division.

**34.** O(n · n!) time at minimum — arguably worse, since `[...remaining.slice(...)]` rebuilds the array at every level, pushing toward O(n² · n!). Space O(n · n!) counting output. The swap-based permutation approach avoids the rebuilding.

**35.** O(n²) time, O(1) space. Transpose then reverse each row = rotate 90° clockwise, in place. Note O(n²) is optimal here since there are n² cells to touch.

**Scoring:** 30+ correct including 10, 17, 18, 23, 26, 29, 30 → Phase 0 is done. Missing the sliding-window/reset-vs-persist ones (23, 28, 29) means Section 8 needs another pass before you move on — those recur in every phase from here.

---

## 20. One-page cheat sheet

```
SIMPLIFICATION
  drop constants          O(3n) → O(n)
  drop lower terms        O(n² + n) → O(n²)
  separate inputs         two arrays → O(n + m), never O(n²)

CURVES (best → worst)
  O(1) < O(log n) < O(√n) < O(n) < O(n log n) < O(n²) < O(n³) < O(2ⁿ) < O(n!)

LOOP SHAPES
  i++                     O(n)
  i *= 2  /  i /= 2       O(log n)
  nested, both n          O(n²)
  nested, inner constant  O(n)
  j = i (triangular)      O(n²)
  two pointers            O(n)
  inner counter RESETS    multiply → O(n²)
  inner counter PERSISTS  add → O(n)

RECURRENCES
  T(n) = T(n-1) + O(1)    O(n)
  T(n) = T(n-1) + O(n)    O(n²)
  T(n) = T(n/2) + O(1)    O(log n)
  T(n) = 2T(n/2) + O(n)   O(n log n)
  T(n) = 2T(n-1) + O(1)   O(2ⁿ)
  memoized                states × work per state

SPACE
  count allocations that scale with n
  + MAX RECURSION DEPTH (the call stack)
  tree recursion → O(h): O(log n) balanced, O(n) skewed
  no tail-call optimization in V8 — tail recursion is still O(n) stack
  V8 stack limit ≈ 10k–15k frames

JS COSTS — O(1)
  arr[i], arr.length, push, pop
  map/set get, set, has, delete, size
  obj.key, key in obj, Object.hasOwn

JS COSTS — O(n)  ⚠️ these look free
  shift, unshift, splice          ← use an index pointer for queues
  includes, indexOf, find         ← use a Set
  slice, concat, [...spread]
  Object.keys / values / entries
  map, filter, reduce, forEach, join
  JSON.parse / stringify

JS COSTS — O(n log n)
  arr.sort()  — TimSort, stable, mutates, string-compares by default
              — × comparator cost; precompute expensive keys

TOP TRAPS
  1  includes/indexOf inside a loop            → Set
  2  {...acc} inside reduce                    → mutate acc
  3  shift() as dequeue                        → index pointer
  4  splice in a loop                          → filter
  5  sort inside a loop                        → hoist it
  6  Object.keys().length in a loop            → Map.size
  7  filter inside map over two arrays         → index into a Map
  8  slice(1) in recursion                     → pass an index
  9  assuming the callback is O(1)             → look inside
  10 string += in a loop                       → array + join
  11 delete on arrays / new Array(n)           → creates holes, deoptimizes

INTERVIEW SCRIPT
  before coding: "brute force is O(n²); I think O(n) with a hash map,
                  trading O(n) space — want the optimal directly?"
  after coding:  "O(n log n), the sort dominates. O(n) space for the copy,
                  or O(1) if I can sort in place."
  always volunteer space complexity unprompted
```

---

## Next

Once you can answer all 35 snippets cold, come back and say **"Start Phase 1"** — arrays and two pointers, where we build the templates you'll reuse for the next nine phases.
