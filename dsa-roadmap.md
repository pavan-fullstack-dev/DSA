# DSA Roadmap — JavaScript / Frontend Interview Track

**Target:** 70 days, ~1–1.5 hrs/day. Language: JavaScript.
**Goal:** solve unseen medium problems out loud, in a shared editor, under time pressure.

---

## How each day works

| Step | Time | What |
|---|---|---|
| 1. Concept | 15 min | Read/watch the pattern. Write the template from memory. |
| 2. Guided | 20 min | 1–2 problems where you already know the pattern applies. |
| 3. Blind | 30 min | 2 problems where you must *recognize* the pattern yourself. |
| 4. Log | 5 min | One line per problem: pattern, key insight, what tripped you. |

**Hard rules**
- Stuck for 25 minutes → read the solution. Then close it and rewrite from scratch. Struggling past 25 min teaches nothing.
- Always state complexity out loud before coding. Interviewers ask every time.
- Re-solve any problem you needed help with after 3 days, then after 10 days.
- Dry-run on paper before running the code. Debugging by clicking "Run" is a red flag in a live interview.

---

## Phase 0 — Foundations (Day 1–2)

Non-negotiable prerequisite. Everything downstream depends on this.

- Big-O: time and space, best/average/worst
- Common curves: O(1), O(log n), O(n), O(n log n), O(n²), O(2ⁿ)
- Amortized analysis (why `array.push` is O(1))
- **JS-specific costs** — this is where JS candidates get caught:
  - `Array.prototype.sort()` → O(n log n), and it mutates
  - `splice` / `shift` / `unshift` → O(n), not O(1)
  - `indexOf` / `includes` on arrays → O(n); on a `Set` → O(1)
  - Spread `[...arr]` → O(n), and easy to accidentally nest inside a loop → O(n²)
  - Strings are immutable → concatenation in a loop is a trap; build an array and `join`
  - `Map` vs plain object: `Map` preserves insertion order, allows any key type, has real O(1) `size`

**Deliverable:** given any snippet, state its complexity in under 10 seconds.

---

## Phase 1 — Arrays & Two Pointers (Day 3–9)

| Day | Topic |
|---|---|
| 3 | Array traversal patterns, in-place modification, `k`-th element |
| 4 | Two pointers — opposite ends (sorted array, palindrome, reverse) |
| 5 | Two pointers — same direction / fast-slow (remove duplicates, move zeroes) |
| 6 | Sliding window — fixed size |
| 7 | Sliding window — variable size (the shrink condition is the whole trick) |
| 8 | Prefix sum & running totals; subarray sum problems |
| 9 | Mixed practice — no hints on which pattern to use |

**Core problems:** Two Sum II, Valid Palindrome, Container With Most Water, 3Sum, Remove Duplicates from Sorted Array, Move Zeroes, Max Sum Subarray of Size K, Longest Substring Without Repeating Characters, Minimum Size Subarray Sum, Subarray Sum Equals K, Product of Array Except Self, Maximum Subarray (Kadane's).

**Signal to recognize:** "sorted array" or "pair/triplet" → two pointers. "Contiguous subarray/substring" → sliding window. "Sum over a range, many queries" → prefix sum.

---

## Phase 2 — Hashing (Day 10–13)

| Day | Topic |
|---|---|
| 10 | `Map` / `Set`, frequency counters |
| 11 | Grouping and bucketing by computed key |
| 12 | Hashing for O(1) lookup to collapse an O(n²) brute force |
| 13 | Mixed practice |

**Core problems:** Two Sum, Valid Anagram, Group Anagrams, Top K Frequent Elements, Contains Duplicate, Longest Consecutive Sequence, Intersection of Two Arrays, First Unique Character, Isomorphic Strings.

Hashing is the single highest-leverage topic. Roughly a third of easy/medium problems reduce to "put it in a Map."

---

## Phase 3 — Sorting & Binary Search (Day 14–19)

| Day | Topic |
|---|---|
| 14 | Sorting algorithms — merge sort and quicksort, implemented by hand |
| 15 | Custom comparators, sorting objects, stability |
| 16 | Binary search on a sorted array — get the template exactly right |
| 17 | Binary search variants — first/last occurrence, insert position, rotated array |
| 18 | Binary search on the *answer space* (not on an array) |
| 19 | Mixed practice |

**Core problems:** Binary Search, Search Insert Position, First Bad Version, Find First and Last Position, Search in Rotated Sorted Array, Find Minimum in Rotated Sorted Array, Koko Eating Bananas, Merge Intervals, Insert Interval, Meeting Rooms, Sort Colors.

Write the binary search template once, memorize it, and stop re-deriving the `mid` and boundary logic. Off-by-one here is the most common live-coding failure.

---

## Phase 4 — Recursion & Backtracking (Day 20–26)

| Day | Topic |
|---|---|
| 20 | Recursion mechanics — base case, recursive case, call stack, stack overflow |
| 21 | Recursion on arrays and strings; tracing a call tree by hand |
| 22 | Divide and conquer |
| 23 | Backtracking template — choose / explore / un-choose |
| 24 | Subsets and combinations |
| 25 | Permutations, handling duplicates |
| 26 | Grid backtracking |

**Core problems:** Reverse String (recursive), Fibonacci with memo, Power of a Number, Subsets, Subsets II, Combination Sum, Permutations, Permutations II, Letter Combinations of a Phone Number, Generate Parentheses, Word Search, N-Queens.

Recursion is also the bridge to trees, graphs, and DP. Do not rush this phase.

---

## Phase 5 — Linked Lists (Day 27–30)

| Day | Topic |
|---|---|
| 27 | Singly linked list, node construction, traversal, dummy-head technique |
| 28 | Reversal (iterative and recursive), fast-slow pointers, cycle detection |
| 29 | Merging, partitioning, reordering |
| 30 | Doubly linked list — needed for LRU cache |

**Core problems:** Reverse Linked List, Middle of the Linked List, Linked List Cycle, Merge Two Sorted Lists, Remove Nth Node From End, Reorder List, Add Two Numbers, Palindrome Linked List, Copy List with Random Pointer.

Lower frequency in frontend interviews than in backend ones, but the pointer manipulation transfers directly to trees, and the doubly linked list is a hard prerequisite for LRU cache — which *is* a frequent frontend question.

---

## Phase 6 — Stacks & Queues (Day 31–35)

| Day | Topic |
|---|---|
| 31 | Stack — use a plain array; matching/validation problems |
| 32 | Monotonic stack — next greater/smaller element |
| 33 | Queue and circular queue; why `shift()` is O(n) and how to avoid it |
| 34 | Deque, min/max stack |
| 35 | Mixed practice |

**Core problems:** Valid Parentheses, Min Stack, Evaluate Reverse Polish Notation, Daily Temperatures, Next Greater Element, Largest Rectangle in Histogram, Implement Queue using Stacks, Sliding Window Maximum, Simplify Path.

---

## Phase 7 — Trees (Day 36–43)

| Day | Topic |
|---|---|
| 36 | Binary tree structure, terminology, recursive traversals (pre/in/post) |
| 37 | Iterative traversals with an explicit stack |
| 38 | BFS / level-order traversal |
| 39 | Depth, height, diameter, balance |
| 40 | BST properties, search, insert, delete |
| 41 | Path problems, lowest common ancestor |
| 42 | Tree construction and serialization |
| 43 | Tries — prefix trees (directly relevant to autocomplete/search UI) |

**Core problems:** Maximum Depth, Invert Binary Tree, Same Tree, Subtree of Another Tree, Level Order Traversal, Right Side View, Validate BST, Kth Smallest in BST, LCA of BST, Diameter of Binary Tree, Balanced Binary Tree, Path Sum, Serialize and Deserialize Binary Tree, Implement Trie.

**Frontend relevance is real here.** The DOM is a tree. A React component tree is a tree. Nested JSON configs, file explorers, comment threads, and category menus are all trees. Expect "flatten this nested structure" or "find a node by id in this JSON tree" as a practical variant.

---

## Phase 8 — Heaps / Priority Queue (Day 44–47)

| Day | Topic |
|---|---|
| 44 | Heap properties, array representation, sift up/down |
| 45 | **Implement MinHeap and MaxHeap from scratch in JS** |
| 46 | Top-K pattern |
| 47 | Two-heap pattern (median), merging k sources |

**Core problems:** Kth Largest Element in an Array, Top K Frequent Elements (heap version), Merge k Sorted Lists, Find Median from Data Stream, Task Scheduler, K Closest Points to Origin.

**JS-specific warning:** JavaScript has no built-in heap or priority queue. Python and Java candidates get one for free; you must write it. Have a 30-line MinHeap memorized, or you will lose 15 minutes of a 45-minute interview building infrastructure.

---

## Phase 9 — Graphs (Day 48–54)

| Day | Topic |
|---|---|
| 48 | Representations — adjacency list vs matrix; building from edge lists |
| 49 | DFS on graphs, visited sets, cycle detection |
| 50 | BFS and shortest path in an unweighted graph |
| 51 | Grid/matrix as an implicit graph (islands, flood fill) |
| 52 | Topological sort (dependency resolution) |
| 53 | Union-Find / Disjoint Set Union |
| 54 | Mixed practice |

**Core problems:** Number of Islands, Clone Graph, Max Area of Island, Pacific Atlantic Water Flow, Surrounded Regions, Rotting Oranges, Course Schedule, Course Schedule II, Number of Connected Components, Graph Valid Tree, Word Ladder.

Skip Dijkstra, Bellman-Ford, and MST unless you're targeting a top-tier algorithmic loop. Master BFS/DFS/topological sort instead — that covers the realistic 95%.

---

## Phase 10 — Dynamic Programming (Day 55–61)

| Day | Topic |
|---|---|
| 55 | Memoization — top-down, derived from recursion you already wrote |
| 56 | Tabulation — bottom-up, and space optimization |
| 57 | 1D DP — climbing stairs, house robber, decode ways |
| 58 | Knapsack family — subset sum, partition, coin change |
| 59 | 2D DP on strings — LCS, edit distance |
| 60 | DP on grids — unique paths, min path sum |
| 61 | Mixed practice |

**Core problems:** Climbing Stairs, House Robber, House Robber II, Coin Change, Longest Increasing Subsequence, Word Break, Partition Equal Subset Sum, Unique Paths, Longest Common Subsequence, Edit Distance, Longest Palindromic Substring, Maximum Product Subarray.

The right mental model: **DP is recursion plus a cache.** Always write the brute-force recursion first, spot the repeated subproblems, then add the memo. Trying to jump straight to a tabulated array is how people get stuck.

For frontend roles, aim for competence on 1D DP and the classic string problems. Don't burn weeks chasing bitmask or tree DP.

---

## Phase 11 — Frontend Implementation Problems (Day 62–66)

**This phase is where frontend interviews are actually won or lost.** Many React interviews replace LeetCode entirely with these.

| Day | Topic |
|---|---|
| 62 | `debounce`, `throttle`, `once`, `memoize` |
| 63 | `deepClone`, `deepEqual`, `flattenObject`, `get`/`set` by path string |
| 64 | Promise utilities — `Promise.all` / `race` / `allSettled` polyfills, promise pool with concurrency limit, retry with exponential backoff, async sequencing |
| 65 | **LRU Cache**, EventEmitter (pub/sub), `curry`, `pipe`/`compose` |
| 66 | Flatten a deeply nested array (all approaches), JSON tree traversal / find node by id, virtualized-list index math, custom `Array.prototype` polyfills (`map`, `filter`, `reduce`) |

Every one of these draws on earlier phases: LRU needs a hash map plus a doubly linked list; deep clone needs recursion plus cycle detection with a `WeakMap`; tree traversal needs DFS. This is the payoff phase.

---

## Phase 12 — Consolidation (Day 67–70)

- Day 67–68: re-solve every problem you flagged in your log. Timed, no hints.
- Day 69: two mock interviews, out loud, on a whiteboard or plain editor with no autocomplete.
- Day 70: build a one-page cheat sheet of templates from memory — two pointers, sliding window, binary search, backtracking, BFS, DFS, MinHeap. If you can't reproduce a template cold, that topic isn't done.

---

## Compressed 3-week track

If an interview lands sooner, do these in order and skip the rest:

1. Complexity + JS-specific costs (1 day)
2. Hashing (2 days)
3. Two pointers + sliding window (3 days)
4. Recursion + backtracking basics (3 days)
5. Trees — traversals, BFS, DFS (3 days)
6. Binary search (1 day)
7. Stacks (1 day)
8. **Frontend implementation problems — all of Phase 11 (4 days)**
9. Mocks (2 days)

Phase 11 stays in even in the compressed version. For a React role it has a higher hit rate than graphs and DP combined.

---

## Resources

| Purpose | Resource |
|---|---|
| Curated problem set | NeetCode 150 (has a topic-ordered roadmap) |
| Pattern recognition | "Grokking the Coding Interview" pattern list |
| Visual intuition | VisuAlgo, Python Tutor (for call-stack tracing) |
| Frontend-specific | GreatFrontEnd, BFE.dev, JSVault |
| Practice | LeetCode — filter by topic tag to match the current phase |

---

## Progress log template

Keep this in a single file and append daily. Reviewing it before an interview is worth more than any last-minute new problem.

```
## Day 07 — Sliding Window (variable)
- Longest Substring Without Repeating Chars | window + Set | shrink from left until valid | 18 min | ✅
- Minimum Size Subarray Sum | window + running sum | forgot to shrink while-loop, used if | 25 min | ⚠️ revisit
- Insight: the shrink condition is what separates fixed from variable windows.
```

---

## Next

Say **"Start Day 1"** and we'll begin with complexity analysis and the JS-specific cost table — including the built-in methods that quietly turn an O(n) solution into O(n²).
