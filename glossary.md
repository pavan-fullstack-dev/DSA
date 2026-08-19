# Glossary — every confusing word in these docs, in plain English

Read this **once, fully**, before any phase document. Every phase doc from here on
assumes these words. If a doc uses a word that isn't here, that's a bug in the doc —
tell me and I'll add it.

Format of each entry:

> **Word** — what it actually means, in normal English.
> *Where you'll see it:* an example from the docs.

---

## Part A — The absolute basics (read this part first)

**Array**
A numbered list of values. `[10, 20, 30]` is an array with 3 values.
The number of a slot is called its **index**, and indexes start at **0**, not 1.
So in `[10, 20, 30]`: index 0 holds 10, index 1 holds 20, index 2 holds 30.

**Index**
The position number of a slot in an array. `arr[2]` means "the value at index 2".

**n**
Just a letter that means "how many items are in the input". If the array has
1,000 items, `n` is 1000. Every complexity statement is written in terms of `n`.

**k**
Another letter, and it always means "some number the problem gives you".
"A window of size k" = "a chunk of k items, where the problem tells you what k is".

**Pointer** (in this course — nothing to do with C/C++ pointers)
A **variable holding an index**. That's all. When a doc says "move the left pointer",
it means `l++` — increase the variable `l` by 1 so it now points at the next slot.

```js
let l = 0;              // "left pointer" — currently pointing at index 0
let r = arr.length - 1; // "right pointer" — pointing at the last index
```

The names you'll see everywhere and what they mean:

| Name | Short for | Means |
|---|---|---|
| `i`, `j` | index | a normal loop counter |
| `l` | left | index of the left end |
| `r` | right | index of the right end |
| `w` | write | the index where the next kept value should be written |
| `lo` / `hi` | low / high | same as left / right |
| `n` | number | the length of the array |
| `k` | — | a size or limit the problem gives you |

**Element**
One value inside the array. `arr[3]` is an element.

**Traverse / traversal**
Walking through the array one element at a time. A `for` loop is a traversal.

**In place**
Change the original array itself instead of building a new one.

```js
// in place — nums itself is modified, nothing new is created
nums[0] = 5;

// NOT in place — a brand new array is created
const copy = nums.map(x => x * 2);
```

Interview problems say "do it in place" when they want you to avoid using extra memory.

**Mutate**
Same idea as "in place": to change something that already exists. `sort()` **mutates** —
it reorders the array you gave it, it does not hand you a new sorted copy.

---

## Part B — Speed words (Big-O)

**Big-O / O(n)**
A way of saying "how much slower does this get when the input gets bigger?"
It ignores exact seconds; it only describes the *shape* of the growth.

| Written | Said out loud | Plain meaning | 1,000 items ≈ |
|---|---|---|---|
| O(1) | "oh one" / constant | same cost no matter how big the input | 1 step |
| O(log n) | "log n" | halving each step | 10 steps |
| O(n) | "oh n" / linear | one pass over the data | 1,000 steps |
| O(n log n) | "n log n" | what a good sort costs | 10,000 steps |
| O(n²) | "n squared" / quadratic | every item compared with every item | 1,000,000 steps |
| O(2ⁿ) | "two to the n" | try every combination — hopeless past n≈30 | astronomical |

The whole point of Phase 1 is: **the obvious answer is O(n²), and there is a trick that
makes it O(n).** That trick is one of four patterns.

**Time complexity** — how many *steps* the code takes.
**Space complexity** — how much extra *memory* the code uses.

**Auxiliary space**
"Extra memory not counting the answer you have to return."
If a problem says "return an array of n results", that result array doesn't count
against you — you were told to produce it. Everything else does.

> Where you'll see it: *"O(1) auxiliary space — the output array doesn't count."*

**Amortized**
"Averaged out over many operations."
`arr.push(x)` is *usually* instant, but occasionally JavaScript has to allocate a
bigger block of memory and copy everything, which is slow. Averaged over many pushes
it's still cheap, so we say push is **O(1) amortized**.

The same word explains why sliding window is fast: a single step *might* do a lot of
work, but across the whole run the total work is still one pass.

**Brute force**
The dumb, obvious solution. Usually "try every possible pair/combination".
You are *supposed* to name it in an interview ("brute force is O(n²)") and then
beat it. Naming it is worth points.

**Constant factor**
The stuff Big-O throws away. O(26n) and O(n) are the *same* complexity, because 26
never grows — the alphabet is always 26 letters. Saying *"O(26n), which is O(n),
because 26 is a constant"* is a thing interviewers listen for.

---

## Part C — The four patterns (the words that name the tricks)

**Pattern / skeleton / template**
A reusable shape of code you memorize once and refill per problem. This course has
four. If you can write the four skeletons from memory, half of Phase 1 is done.

**Two pointers**
Two index variables walking through the array instead of two nested loops.
Two flavours:
- **Opposite ends** — one starts at the left, one at the right, they walk toward
  each other and meet in the middle.
- **Same direction** — both start on the left; one runs ahead, one trails behind.

**Write pointer** (also called slow/fast, or read/write pointer)
The "same direction" flavour. One pointer **reads** every element; a second pointer
marks **where the next keeper goes**. Used to delete/move items without creating a
new array.

**Window** ← *the word you asked about*

A **window** is simply **a continuous chunk of the array**, described by where it
starts and where it ends. Nothing more.

```
arr =  [ 4 , 2 , 9 , 7 , 1 , 5 ]
index    0   1   2   3   4   5

              └───────────┘
              the window from index 1 to index 3, i.e. [2, 9, 7]
```

You track it with two variables — `l` (left edge) and `r` (right edge). The values
*inside* `l..r` are "in the window"; everything else is outside it.

**Sliding window**
Moving that chunk along the array without rebuilding it each time.

```
step 1:  [ 4 , 2 , 9 ] 7   1   5      window = indexes 0..2
step 2:    4 [ 2 , 9 , 7 ] 1   5      window = indexes 1..3
step 3:    4   2 [ 9 , 7 , 1 ] 5      window = indexes 2..4
```

The key move: to go from step 1 to step 2 you don't re-add everything. You **add the
one element that entered on the right and subtract the one that left on the left**.
That's why it's fast.

Think of it as a physical window frame you slide along a row of houses: you only ever
gain one house and lose one house.

- **Fixed window** — the chunk is always exactly `k` wide.
- **Variable window** — the chunk grows and shrinks; you're looking for the longest
  or shortest chunk that satisfies some rule.

**Prefix sum**
A running total. `prefix[i]` = "the sum of everything before position i".

```
nums    =  [ 1 ,  2 ,  3 ,  4 ]
prefix  = [0,  1,   3,   6,  10]
            ↑   ↑    ↑    ↑    ↑
          sum of first 0, 1, 2, 3, 4 items
```

Why it's useful: the sum of any chunk = **one subtraction**.
Sum of `nums[1..2]` (= 2+3 = 5) is `prefix[3] - prefix[1]` = 6 − 1 = 5. Instant.

**Difference array**
The reverse of prefix sum. Instead of answering many "what's the sum of this range"
questions fast, it applies many "add 5 to everything in this range" updates fast.

**Kadane's algorithm**
A one-line trick for "biggest sum of a continuous chunk". At each element you ask one
question: *is the best chunk ending here just this element alone, or this element
glued onto the best chunk ending at the previous position?*

**Index-as-hash / cyclic sort**
A trick that only works when the values in the array happen to be numbers in the range
1..n. Then value `7` "belongs" at index 6, so the array can be used as its own lookup
table — no extra memory needed.

---

## Part D — Words describing problem statements

**Contiguous**
Side by side, no gaps. `[2, 9, 7]` is contiguous inside `[4, 2, 9, 7, 1]`.
`[4, 9, 1]` is **not** — you skipped items.

**Subarray**
A **contiguous** chunk of an array. `[2, 9, 7]` from the example above.
An array of length n has n(n+1)/2 subarrays — that's why checking them all is O(n²).

**Substring**
The same thing, but for a string. `"ell"` is a substring of `"hello"`.

**Subsequence**
Items in order but **allowed to skip**. `[4, 9, 1]` is a subsequence of
`[4, 2, 9, 7, 1]`. `"hlo"` is a subsequence of `"hello"`.

> **This one word decides your whole approach.** Subarray/substring → sliding window
> is on the table. Subsequence → windows are useless, it's a Dynamic Programming
> problem (Phase 10). Read the problem statement for this word specifically.

**Sorted**
Arranged in order, smallest to largest. Huge amounts of Phase 1 only work on sorted
input, because sortedness lets you *rule out* whole regions without checking them.

**Monotone / monotonic**
"Only moves one way — never flips back."
A window's validity is **monotone** if adding more items can only ever make it worse
(never suddenly better again).

Example: with all-positive numbers, adding an item always makes the sum bigger.
That's monotone → sliding window works.
With negative numbers allowed, adding an item might make the sum *smaller*.
Not monotone → sliding window silently gives wrong answers.

**Invariant**
A promise that is true at every single step of the loop.

> Example: in the write-pointer pattern, the invariant is *"`arr[0..w-1]` is always the
> correct answer for everything processed so far."*

Interviewers give real credit for stating invariants out loud, because it proves you
understand the loop instead of having memorized it.

**Predicate**
A yes/no test. `x => x !== 0` ("is this not zero?") is a predicate.

**Dedup / deduplication**
Removing duplicates from the answer.

**Edge case**
An unusual input that breaks naive code: empty array, one element, all values the
same, all negative numbers.

**Dry run**
Tracing your code **on paper**, line by line, writing down what each variable holds at
each step — before you click Run. In a live interview, debugging by repeatedly hitting
Run looks bad; a dry run looks senior.

**Off-by-one**
The classic bug where a loop runs one time too many or too few, or you read `arr[i+1]`
when you meant `arr[i]`. The single most common failure in this phase.

---

## Part E — JavaScript and browser words

**V8** ← *the other word you asked about*

**V8 is the program that actually runs your JavaScript.** It's Google's JavaScript
engine — the piece of C++ inside Chrome, Edge, and Node.js that reads your JS and
executes it.

That's the whole definition. When a doc says *"a V8 gotcha"*, it means:

> "This is a quirk of how JavaScript actually runs, which a Python or Java candidate
> in the same interview would not hit. Nothing to do with the algorithm — everything
> to do with the language."

Other engines exist (JavaScriptCore in Safari, SpiderMonkey in Firefox), but they
behave the same way for everything in these docs. So mentally read **"V8" as "the
JavaScript engine"** and you'll never be confused by it again.

Why the docs mention V8 at all: to warn you about places where JavaScript behaves
differently from what algorithm textbooks (written for C++/Java/Python) assume.
For example, `%` on a negative number gives a different answer in JS than in Python,
and that difference will silently break a correct-looking solution.

**Engine**
Same thing as above — the program that runs JS. V8 is one engine.

**Hash map / hash table**
A lookup table. Give it a key, it gives you back a value, instantly (O(1)).
In JavaScript this is `Map` (or a plain object `{}`).

```js
const m = new Map();
m.set('a', 1);
m.get('a');    // 1 — instant, no searching
```

**Set**
Like a Map, but it only stores keys, not values. Use it to answer "have I seen this
before?" instantly.

```js
const s = new Set();
s.add('x');
s.has('x');   // true — instant
```

**Hole** (in an array)
A slot that doesn't exist at all, as opposed to a slot holding `undefined`.
`new Array(3)` creates 3 holes. Holey arrays are slower and behave strangely
(`map` skips holes). The fix is always `new Array(3).fill(0)`.

**Charset / alphabet**
The set of possible characters. For lowercase English it's 26. Counting arrays of
size 26 are O(1) space *because 26 never grows*.

**Immutable**
Cannot be changed after creation. **Strings in JavaScript are immutable** —
`s[0] = 'x'` silently does nothing. To "change" a string you build a new one, which
costs O(n). This is why there's no O(1)-space string reversal in JS.

**LC 15**
"LeetCode problem number 15." The docs use LC numbers so you can search the exact
problem on leetcode.com. LC 15 is 3Sum.

---

## Part F — Reading the code shorthand

| You see | It means |
|---|---|
| `w++` | use `w`, then add 1 to it |
| `++w` | add 1 to `w` first, then use it |
| `arr[w++] = x` | write `x` at index `w`, **then** move `w` forward one |
| `r - l + 1` | the **length** of the window from `l` to `r` (the `+1` is because both ends are included) |
| `[a[i], a[j]] = [a[j], a[i]]` | swap two elements |
| `sum += arr[i] - arr[i-k]` | slide the window: add the newcomer, subtract the leaver |
| `(a, b) => a - b` | "sort numerically, ascending" — always required for numbers |
| `Math.floor(x)` | round down |
| `Infinity` | a starting value for "best minimum" — anything real beats it |
| `-Infinity` | a starting value for "best maximum" — anything real beats it |
| `nums[i] ?? 0` | "the value, or 0 if it's missing" |
| `map.get(x) \|\| 0` | "the count so far, or 0 if never seen" |

**Why `r - l + 1` and not `r - l`:** if `l = 2` and `r = 2`, the window holds exactly
one element. `r - l` gives 0 (wrong), `r - l + 1` gives 1 (right).
Check it that way every single time.

---

## Part G — What the 70-day plan actually is, in one paragraph

There are ~15 recurring tricks in coding interviews. The plan spends 70 days
introducing them in dependency order (each phase needs the previous one), because a
"medium" interview problem is almost never original — it's one of those tricks wearing
a costume. **Phase 1 (this one) covers 4 of the highest-frequency tricks.** The daily
routine — 15 min reading, 20 min guided problems, 30 min unassisted problems, 5 min
logging — exists so that you practise *recognizing which trick applies*, which is the
actual skill being tested. Solving a problem you were told the trick for teaches you
almost nothing; recognizing the trick cold is the whole game.

---

## Where to go next

| File | What it's for |
|---|---|
| `phase-1-arrays-two-pointers.md` | **All of Phase 1, one file, plain English** — the four patterns, worked walkthroughs, JS traps, 7-day schedule, 32 practice problems with answers, cheat sheet. |
| `dsa-roadmap.md` | The 70-day schedule. |
