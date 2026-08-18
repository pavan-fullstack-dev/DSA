# DSA Prep — JavaScript / Frontend Interview Track

Personal DSA study repo. Docs are generated one phase at a time, then studied before moving on.

---

## Context (paste this into a new chat to continue)

> I'm a Senior Full Stack Developer with ~6 years of experience. My stack is React.js, React Native, TypeScript, Node.js, TanStack Query, and Playwright. I'm preparing for frontend/React developer interviews and working through DSA in **JavaScript only** — not Python, not C++.
>
> I'm following a 70-day, 12-phase DSA roadmap tailored to frontend interviews. It's deliberately weighted toward arrays, strings, hashmaps, recursion, and trees, lighter on advanced graphs and DP, and includes a dedicated phase on JS implementation problems (debounce, throttle, deep clone, LRU cache, promise pool, `Promise.all` polyfill) because those come up more often in React loops than graph algorithms do.
>
> **How I want to work:** one phase at a time. You give me a single exhaustive `.md` document for that phase — complete enough that I don't need to come back with follow-up questions. I study it, then return and ask for the next phase.
>
> **What I want in each phase doc:**
> - Full conceptual explanation, not summaries
> - All code examples in JavaScript
> - Reusable templates I can memorize (the pattern skeleton, stated once, cleanly)
> - Explicit time and space complexity for every example
> - JS/V8-specific gotchas — where JS candidates get caught vs Python/Java candidates
> - Frontend relevance called out where it exists (the DOM is a tree, nested JSON configs, etc.)
> - Named LeetCode problems to practice per topic
> - A practice set at the end with worked answers
> - Interview phrasing — how to actually say the answer out loud
> - A one-page cheat sheet at the end
>
> **Style:** direct, no filler. Correct me when I'm wrong rather than hedging. Include the honest/nuanced answer where the textbook answer is misleading (e.g. string `+=` is not really O(n²) in V8 because of rope structures — I want that kind of detail, not the simplified version).
>
> **Completed so far:** Phase 0 (complexity analysis). See `phase-0-complexity-analysis.md` in this repo.
>
> **Next up:** Phase 1 — Arrays & Two Pointers.

---

## Roadmap — 12 phases, 70 days

| Phase | Days | Topic | Status |
|---|---|---|---|
| 0 | 1–2 | Foundations — Big-O, space complexity, amortized analysis, JS cost table | ✅ Done |
| 1 | 3–9 | Arrays & Two Pointers — sliding window, prefix sum | ⬜ Next |
| 2 | 10–13 | Hashing — Map/Set, frequency counters, grouping | ⬜ |
| 3 | 14–19 | Sorting & Binary Search — templates, variants, search on answer space | ⬜ |
| 4 | 20–26 | Recursion & Backtracking — subsets, permutations, grid | ⬜ |
| 5 | 27–30 | Linked Lists — reversal, fast/slow, doubly linked | ⬜ |
| 6 | 31–35 | Stacks & Queues — monotonic stack, deque | ⬜ |
| 7 | 36–43 | Trees — traversals, BFS, BST, LCA, tries | ⬜ |
| 8 | 44–47 | Heaps — implement MinHeap from scratch (JS has none built in), top-K | ⬜ |
| 9 | 48–54 | Graphs — BFS/DFS, grids, topological sort, union-find | ⬜ |
| 10 | 55–61 | Dynamic Programming — memo, tabulation, 1D, knapsack, strings, grids | ⬜ |
| 11 | 62–66 | **Frontend implementation problems** — debounce, deep clone, LRU, promise pool | ⬜ |
| 12 | 67–70 | Consolidation — re-solve flagged problems, mocks, template cheat sheet | ⬜ |

Full detail per phase is in `dsa-roadmap.md`.

### Compressed 3-week fallback

If an interview lands early: complexity → hashing → two pointers/sliding window → recursion basics → trees → binary search → stacks → **all of Phase 11** → mocks. Phase 11 stays in even when compressed; for a React role it has a higher hit rate than graphs and DP combined.

---

## Repo structure

```
/
├── README.md                          ← this file
├── dsa-roadmap.md                     ← the full 70-day plan
├── phase-0-complexity-analysis.md     ← ✅ complete
├── phase-1-arrays-two-pointers.md     ← to generate
├── ...
├── solutions/                         ← my own solution files, one per problem
└── log.md                             ← daily progress log
```

---

## Daily routine

| Step | Time | What |
|---|---|---|
| Concept | 15 min | Read the pattern. Write the template from memory. |
| Guided | 20 min | 1–2 problems where I already know the pattern applies. |
| Blind | 30 min | 2 problems where I have to recognize the pattern myself. |
| Log | 5 min | One line per problem in `log.md`. |

**Rules**
- Stuck 25 minutes → read the solution, then close it and rewrite from scratch.
- State complexity out loud before coding.
- Re-solve anything I needed help with after 3 days, then after 10 days.
- Dry-run on paper before hitting Run.

### Log format

```
## Day 07 — Sliding Window (variable)
- Longest Substring Without Repeating Chars | window + Set | shrink left until valid | 18 min | ✅
- Minimum Size Subarray Sum | window + running sum | used if instead of while to shrink | 25 min | ⚠️ revisit
- Insight: the shrink condition is what separates fixed from variable windows.
```

---

## Phase 0 — key takeaways to keep loaded

Carry these forward into every later phase.

**The resets-vs-persists test.** If an inner loop's counter resets each outer iteration → multiply → O(n²). If it persists and only moves forward → add → O(n). This is why sliding window is linear despite the nested `while`, and calling it O(n²) is a costly interview mistake.

**JS methods that look O(1) but are O(n):** `shift`, `unshift`, `splice`, `includes`, `indexOf`, `find`, `slice`, `concat`, `[...spread]`, `Object.keys`.

**`shift()` as dequeue turns BFS from O(V+E) into O(V²+E).** Use an index pointer: `let head = 0; ... queue[head++]`.

**Spread inside `reduce` is O(n²).** `items.reduce((acc, x) => ({...acc, [x.id]: x}), {})` — ubiquitous in React/Redux code.

**Space complexity includes the call stack.** Tree recursion is O(h): O(log n) balanced, O(n) skewed. V8 has no tail-call optimization, so tail recursion is still O(n) stack.

**Recurrences worth knowing cold:** `T(n)=T(n/2)+O(1)` → O(log n). `T(n)=2T(n/2)+O(n)` → O(n log n). `T(n)=2T(n-1)+O(1)` → O(2ⁿ). Memoized recursion → states × work per state.

**`sort()` mutates, is TimSort (stable, O(n log n), O(n) best case on sorted input), and string-compares by default** — always pass `(a, b) => a - b` for numbers.

**JS has no built-in heap or priority queue.** Phase 8 requires implementing MinHeap from scratch; have ~30 lines memorized.

---

## Interview framing (reuse in every phase)

**Before coding:**
> "Brute force here is O(n²) — check every pair. I think we can get to O(n) with a hash map by trading O(n) space. Want me to go straight to the optimal version, or start with brute force and improve it?"

**After coding:**
> "Time is O(n log n) — the sort dominates, the scan after is O(n). Space is O(n) for the copy, or O(1) if I can sort in place."

Always volunteer space complexity unprompted. Most candidates wait to be asked.

---

## Resources

| Purpose | Resource |
|---|---|
| Curated problem set | NeetCode 150 |
| Pattern recognition | Grokking the Coding Interview pattern list |
| Visual intuition | VisuAlgo, Python Tutor (call-stack tracing) |
| Frontend-specific | GreatFrontEnd, BFE.dev, JSVault |
| Practice | LeetCode, filtered by the current phase's topic tag |