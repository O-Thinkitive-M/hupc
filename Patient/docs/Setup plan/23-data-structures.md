# 23 — Data Structures & Algorithms (Reusable Logic)

Harmony EMR (HUPC) frontend foundation. **React 19 + TS strict (`noUncheckedIndexedAccess`), Vite 8, MUI v7.**
**Shared verbatim** across Admin / Provider / Patient / Website-Widget portals.

> **Thesis:** Pick the data structure that makes the operation **O(1) and the call-site a one-liner**, and
> pick the **algorithm that solves complex logic correctly and fast** — then expose both as **reusable,
> typed helpers** so they are easy to use *anywhere*. Never scatter ad-hoc `for` loops, repeated
> `.find()` / `.filter()` scans, deeply-nested `if` chains, or hand-rolled boilerplate across features —
> that is the exact "normal logic" this file exists to replace. Same rule as the config-driven-primitive
> thesis: solve the shape once, reuse it everywhere. **Right structure + right algorithm = less code, fewer
> branches, faster runtime.**

---

## 1. The core rule — match the structure to the access pattern

Before writing logic, ask **"what is the hot operation?"** then pick the structure whose cost for that
operation is constant:

| Use case (what you actually do) | ✅ Right structure | ❌ Boilerplate / "normal logic" to avoid | Why |
|---|---|---|---|
| Look up an entity **by id** | `Map<Id, T>` / normalized `Record` (`indexBy`) | `arr.find(x => x.id === id)` everywhere | `find` is O(n) per call → O(n²) in a render loop |
| "Is this **selected / in set**?" / dedup | `Set<Id>` | `arr.includes(id)` / `arr.indexOf` | `includes` is O(n); `Set.has` is O(1) |
| **Group** rows by a key (status, day, provider) | `Map<K, T[]>` (`groupBy`) | nested loops + manual bucket objects | one pass, typed, no `undefined` bucket bugs |
| **Ordered unique** list | `Set` → spread to array | `filter((v,i,a)=>a.indexOf(v)===i)` | O(n) vs O(n²), preserves insertion order |
| **Count / frequency** | `Map<K, number>` (`countBy`) | object with `obj[k] = (obj[k]||0)+1` | typed keys, no prototype-key pitfalls |
| Fixed **enum → label / color / icon** | `Record<Enum, Meta>` registry | `switch` / `if-else` chains | data not control-flow; see `04-config-registries.md` |
| **Hierarchy** (nav tree, org units, threaded notes) | tree: `{ id, children: Node[] }` + index map | recursive ad-hoc traversal each time | one `walkTree` / `findInTree` helper, reused |
| **FIFO** (toast queue, request queue) | array as queue (`push` / `shift`) or ring buffer | manual index juggling | clear intent, bounded variants easy |
| **LIFO** (undo, modal stack, breadcrumb trail) | stack (array `push` / `pop`) | scattered "remember previous" vars | single source of truth for the stack |
| **Bounded cache** (recently-viewed patients) | `LRUCache` (Map insertion order) | growing object never evicted | memory-safe, O(1) get/set/evict |
| **Many-to-many / adjacency** (role→perms) | `Map<K, Set<V>>` | array of pairs + filter | O(1) membership + O(1) add |
| **Sorted range / nearest** lookup | sorted array + binary search | linear scan + `Math.min` | O(log n) vs O(n) on large slot lists |

> Rule of thumb: **lookup / membership / grouping ⇒ `Map` / `Set` / `Record`.** Reach for a plain array
> only when you genuinely need an **ordered, index-addressed sequence** you iterate start-to-end.

---

## 2. The reusable toolkit (`src/lib/ds/`)

One small, **fully-typed, dependency-free** module. Import the helper instead of re-writing the loop. Every
function is pure and tree-shakeable.

```ts
// src/lib/ds/collections.ts  — pure, typed, reusable everywhere

/** Index a list by a key → O(1) lookup map (replaces repeated .find). */
export function indexBy<T, K extends PropertyKey>(items: readonly T[], key: (t: T) => K): Map<K, T> {
  const m = new Map<K, T>();
  for (const it of items) m.set(key(it), it);
  return m;
}

/** Group a list into buckets by key (one pass, no manual bucket init). */
export function groupBy<T, K extends PropertyKey>(items: readonly T[], key: (t: T) => K): Map<K, T[]> {
  const m = new Map<K, T[]>();
  for (const it of items) {
    const k = key(it);
    const bucket = m.get(k);
    if (bucket) bucket.push(it);
    else m.set(k, [it]);
  }
  return m;
}

/** Frequency count → Map<key, count>. */
export function countBy<T, K extends PropertyKey>(items: readonly T[], key: (t: T) => K): Map<K, number> {
  const m = new Map<K, number>();
  for (const it of items) m.set(key(it), (m.get(key(it)) ?? 0) + 1);
  return m;
}

/** Ordered de-dup (insertion order preserved). */
export const unique = <T>(items: readonly T[]): T[] => [...new Set(items)];

/** Split into [pass, fail] in one pass. */
export function partition<T>(items: readonly T[], pred: (t: T) => boolean): [T[], T[]] {
  const pass: T[] = [], fail: T[] = [];
  for (const it of items) (pred(it) ? pass : fail).push(it);
  return [pass, fail];
}

/** Build a Set keyed from a list — O(1) membership tests. */
export const toSet = <T, K>(items: readonly T[], key: (t: T) => K): Set<K> =>
  new Set(items.map(key));
```

```ts
// src/lib/ds/lru.ts — bounded cache (recently-viewed, memoized lookups)
export class LRUCache<K, V> {
  private map = new Map<K, V>();
  constructor(private max: number) {}
  get(k: K): V | undefined {
    const v = this.map.get(k);
    if (v !== undefined) { this.map.delete(k); this.map.set(k, v); } // touch → most-recent
    return v;
  }
  set(k: K, v: V): void {
    if (this.map.has(k)) this.map.delete(k);
    this.map.set(k, v);
    if (this.map.size > this.max) this.map.delete(this.map.keys().next().value as K); // evict oldest
  }
  has(k: K): boolean { return this.map.has(k); }
}
```

```ts
// src/lib/ds/tree.ts — hierarchy traversal (nav, org units, threaded notes)
export interface TreeNode<T> { value: T; children: TreeNode<T>[]; }

export function walkTree<T>(roots: readonly TreeNode<T>[], visit: (v: T, depth: number) => void, depth = 0): void {
  for (const n of roots) { visit(n.value, depth); walkTree(n.children, visit, depth + 1); }
}

export function findInTree<T>(roots: readonly TreeNode<T>[], pred: (v: T) => boolean): T | undefined {
  for (const n of roots) {
    if (pred(n.value)) return n.value;
    const hit = findInTree(n.children, pred);
    if (hit !== undefined) return hit;
  }
  return undefined;
}
```

> **`noUncheckedIndexedAccess` note:** `Map.get` / array index return `T | undefined`. The helpers above
> handle it explicitly (`?? 0`, `if (bucket)`). At call-sites, always narrow the `undefined` before use —
> never assert with `!`.

---

## 3. Algorithms for complex logic (`src/lib/algo/`)

When the task is **not just a lookup** — searching, sorting, matching, scheduling, traversing, rate-limiting
— reach for the **named algorithm**, not a pile of nested `if`s. Each lives once in `src/lib/algo/`, typed
and tested, and is **paired with the structure it operates on** (§1–2). Picking the right algorithm removes
"unwanted conditions" and turns O(n²)/O(n) hot paths into O(log n)/O(1).

| Complex logic (use case) | ✅ Algorithm to use | ❌ Boilerplate / unwanted conditions to avoid | Cost |
|---|---|---|---|
| Find in a **sorted** list (slot at/after a time) | **Binary search** | linear `for` + `if (t >= x) break` | O(log n) vs O(n) |
| Order rows by N keys (status, then date) | **Stable sort + composed comparator** | chained `if/else if` comparisons inline | clear, reusable |
| Merge two sorted streams (calendar feeds) | **Two-pointer / merge** | concat + full re-sort | O(n+m) vs O(n log n) |
| Detect **overlapping appointments** / free slots | **Sweep-line on sorted intervals** | nested loop comparing every pair | O(n log n) vs O(n²) |
| Type-ahead / search box | **Debounce** (+ optional throttle for scroll) | fire a request per keystroke | fewer calls, no races |
| Expensive derived value recomputed often | **Memoization** (`useMemo` / `memoize` by key) | recompute every render | O(1) on cache hit |
| Fuzzy / ranked client search | **Scoring + top-K (partial sort / heap)** | `filter` + `includes` then full sort | only rank what matches |
| Walk a graph/dependency (perms, referrals) | **BFS / DFS** with a `visited` `Set` | recursion with no visited guard → cycles | linear, cycle-safe |
| Paginate / windowed render of huge lists | **Windowing / virtualization math** | render all rows, slice in template | bounded DOM (see `14`) |
| Retry a flaky request | **Exponential backoff** | fixed-interval `setInterval` loop | fewer collisions |
| Compare old/new list to patch UI/state | **Keyed diff** (`Map` by id) | full re-render / index-based diff | minimal updates |
| Batch rapid events (autosave, audit flush) | **Throttle / coalesce queue** | write on every change | fewer writes |

```ts
// src/lib/algo/binarySearch.ts — first index whose value >= target (lower-bound)
export function lowerBound<T>(sorted: readonly T[], target: number, valueOf: (t: T) => number): number {
  let lo = 0, hi = sorted.length;            // [lo, hi)
  while (lo < hi) {
    const mid = (lo + hi) >>> 1;
    if (valueOf(sorted[mid]!) < target) lo = mid + 1;
    else hi = mid;
  }
  return lo;                                  // == length if none qualify
}
```

```ts
// src/lib/algo/comparator.ts — compose multi-key sorts, no if-chains
type Cmp<T> = (a: T, b: T) => number;
export const byKey = <T>(sel: (t: T) => number | string, dir: 'asc' | 'desc' = 'asc'): Cmp<T> =>
  (a, b) => { const x = sel(a), y = sel(b); const c = x < y ? -1 : x > y ? 1 : 0; return dir === 'asc' ? c : -c; };
/** Apply comparators in priority order; first non-zero wins. */
export const sortBy = <T>(...cmps: Cmp<T>[]): Cmp<T> =>
  (a, b) => { for (const c of cmps) { const r = c(a, b); if (r) return r; } return 0; };
// usage: rows.slice().sort(sortBy(byKey(r => r.statusRank), byKey(r => r.date, 'desc')))
```

```ts
// src/lib/algo/intervals.ts — overlap detection via sweep (appointments, room booking)
export interface Interval { start: number; end: number; }
export function hasOverlap(intervals: readonly Interval[]): boolean {
  const sorted = [...intervals].sort((a, b) => a.start - b.start);
  for (let i = 1; i < sorted.length; i++) if (sorted[i]!.start < sorted[i - 1]!.end) return true;
  return false;
}
```

```ts
// src/lib/algo/rate.ts — debounce + throttle (search, scroll, autosave)
export function debounce<A extends unknown[]>(fn: (...a: A) => void, ms: number) {
  let t: ReturnType<typeof setTimeout> | undefined;
  return (...a: A) => { clearTimeout(t); t = setTimeout(() => fn(...a), ms); };
}
export function throttle<A extends unknown[]>(fn: (...a: A) => void, ms: number) {
  let last = 0, t: ReturnType<typeof setTimeout> | undefined, lastArgs: A;
  return (...a: A) => {
    lastArgs = a; const now = performance.now(); const wait = ms - (now - last);
    if (wait <= 0) { last = now; fn(...a); }
    else if (!t) t = setTimeout(() => { last = performance.now(); t = undefined; fn(...lastArgs); }, wait);
  };
}

/** Memoize a pure fn by a string key (bound by LRU to stay memory-safe). */
export function memoize<A extends unknown[], R>(fn: (...a: A) => R, keyOf: (...a: A) => string, max = 100) {
  const cache = new LRUCache<string, R>(max);              // from src/lib/ds/lru.ts
  return (...a: A): R => {
    const k = keyOf(...a); const hit = cache.get(k);
    if (hit !== undefined) return hit;
    const v = fn(...a); cache.set(k, v); return v;
  };
}
```

```ts
// src/lib/algo/graph.ts — BFS over an adjacency Map (perms, referral chains)
export function bfs<N>(start: N, neighbors: (n: N) => Iterable<N>): N[] {
  const seen = new Set<N>([start]); const queue: N[] = [start]; const order: N[] = [];
  while (queue.length) {
    const n = queue.shift()!; order.push(n);
    for (const next of neighbors(n)) if (!seen.has(next)) { seen.add(next); queue.push(next); }
  }
  return order;                                            // visited Set ⇒ cycle-safe
}
```

> **React integration:** wrap `debounce`/`throttle` in a stable ref (or `useMemo`) so the timer survives
> re-renders; feed the debounced value into a **stable query key** (`07-state-and-data.md`). Heavy
> pure computations (sort + filter + score over large rows) go in `useMemo` keyed on the inputs — the
> memoized algorithm runs only when inputs actually change.

### How to choose (decision order)
1. **Is it a lookup/membership/group?** → use a structure from §1–2, no algorithm needed.
2. **Is the input sorted (or cheap to keep sorted)?** → binary search / two-pointer / merge, not a scan.
3. **Is it called rapidly from UI events?** → debounce / throttle / memoize before optimizing the body.
4. **Is it a traversal?** → BFS/DFS with a `visited` `Set`; never recurse without a cycle guard.
5. **Only if none fit** → write explicit logic, but isolate it as one tested helper in `src/lib/algo/`.

> **Don't over-engineer:** for a list of ~10 items a plain `.find()` is fine — reach for binary search,
> heaps, or sweep-line when N is large or the path is hot. Measure against the budgets in `14-performance-scalability.md`.

---

## 4. Normalized entity state — the canonical collection shape

For any **collection of entities** (patients, appointments, users) held in a store or derived from a list
response, store it **normalized**: a lookup map + an order array. This is the single pattern for "a list I
also need to address by id."

```ts
// shape
interface Normalized<T> { byId: Record<string, T>; allIds: string[]; }

// build from an API list response (one pass)
export function normalize<T extends { id: string }>(list: readonly T[]): Normalized<T> {
  const byId: Record<string, T> = {};
  const allIds: string[] = [];
  for (const it of list) { byId[it.id] = it; allIds.push(it.id); }
  return { byId, allIds };
}

// O(1) read, O(n) ordered iteration
const patient = state.byId[id];                       // direct, no scan
const ordered = state.allIds.map((id) => state.byId[id]);
```

| Operation | Normalized cost | Plain-array cost |
|---|---|---|
| Get by id | **O(1)** | O(n) `find` |
| Update one | **O(1)** | O(n) `map` rebuild |
| Keep order | `allIds` | native |
| Membership | `id in byId` O(1) | O(n) |

> Pairs with `07-state-and-data.md`: TanStack Query owns the **server copy**; normalize only when you need
> id-addressing in a Zustand store or a heavy `DataTable` selection model. Don't duplicate server data into
> Zustand just to normalize it — derive with `useMemo(() => indexBy(rows, r => r.id), [rows])` at the edge.

---

## 5. Immutability with Map / Set / normalized state

These structures are **mutable** — that breaks React/Zustand/Query change detection if mutated in place.
Always write a **new** container:

```ts
// Zustand — copy then mutate the copy
addSelected: (id) => set((s) => { const next = new Set(s.selected); next.add(id); return { selected: next }; }),

// normalized update — new object, new map entry
upsert: (p) => set((s) => ({ byId: { ...s.byId, [p.id]: p }, allIds: s.byId[p.id] ? s.allIds : [...s.allIds, p.id] })),
```

> **Never** mutate the TanStack Query cache object directly — invalidate or `setQueryData` with a new value
> (see `07`). Helpers in §2–3 already return fresh containers (they build, never mutate inputs).

---

## 6. Complexity cheat-sheet (pick with eyes open)

| Operation | Array | Object `Record` | `Map` | `Set` |
|---|---|---|---|---|
| Access by key | O(n) `find` | O(1) | O(1) | — |
| Membership | O(n) `includes` | O(1) `in` | O(1) `has` | **O(1) `has`** |
| Insert | O(1) `push` | O(1) | O(1) | O(1) |
| Delete by key | O(n) | O(1) `delete` | O(1) `delete` | O(1) |
| Ordered iterate | ✅ native | insertion order | **insertion order** | insertion order |
| Non-string keys | — | ❌ (coerced) | ✅ any type | ✅ any type |
| Size | `.length` | `Object.keys().length` | `.size` O(1) | `.size` O(1) |

**Algorithm costs:** linear scan **O(n)** · binary search **O(log n)** · sort **O(n log n)** · pairwise
nested loop **O(n²)** (avoid) · BFS/DFS **O(V+E)** · memoized hit **O(1)**.

> Prefer **`Map`/`Set`** over plain objects when keys are dynamic, non-string, or you need `.size` /
> ordered iteration. Prefer **`Record`** for **fixed, known** key sets (enums, config registries).

---

## 7. Where things live (naming & placement)

| Kind | Location | Example |
|---|---|---|
| Generic structures & helpers | `src/lib/ds/` | `indexBy`, `groupBy`, `LRUCache`, `walkTree` |
| Generic algorithms | `src/lib/algo/` | `lowerBound`, `sortBy`, `hasOverlap`, `debounce`, `bfs`, `memoize` |
| Enum → metadata registries | `src/config/` | status/color/label maps (see `04`) |
| Feature-specific indexes | `features/<f>/` (memoized) | `useMemo(() => indexBy(rows, r => r.id), [rows])` |
| Normalized store slices | `src/store/` | `byId` / `allIds` slices (see `07`) |

> If a structure, algorithm, or helper is used by **two or more features**, it belongs in `src/lib/ds/` or
> `src/lib/algo/` — not copied. One implementation, fully typed, tested once (`15-testing.md`).

---

## Do / Don't

| ✅ Do | ❌ Don't |
|---|---|
| `indexBy(rows, r => r.id).get(id)` | `rows.find(r => r.id === id)` inside a render/loop |
| `new Set(ids).has(id)` | `ids.includes(id)` in a hot path |
| `groupBy(appts, a => a.day)` | manual `acc[day] = acc[day] || []` boilerplate |
| `Record<Status, Meta>` lookup | `switch (status)` returning labels/colors |
| `lowerBound(slots, t, s => s.start)` on sorted slots | linear `for` loop with `if (t >= x) break` |
| `rows.sort(sortBy(byKey(...), byKey(...)))` | chained `if/else if` comparisons inline |
| `hasOverlap(intervals)` (sweep) | nested loop comparing every appointment pair |
| `debounce(search, 300)` / `memoize(fn, keyOf)` | a request per keystroke / recompute every render |
| `bfs(start, neighbors)` with a `visited` `Set` | recursion with no visited guard (cycles, stack blow-up) |
| Import the shared helper / algorithm | re-paste the same loop or nested `if`s in a new feature |
| Return a new Map/Set on write | `set.add(x)` on state then `set({ set })` |

---

## Golden rules
- Choose the structure by the **hot operation**: lookup/membership/grouping ⇒ `Map`/`Set`/`Record`, never an O(n) array scan.
- Choose the **named algorithm** for complex logic (search/sort/merge/overlap/traversal/rate-limit) — never nested `if`s or O(n²) loops where O(log n)/O(n) exists.
- Put generic structures in **`src/lib/ds/`** and generic algorithms in **`src/lib/algo/`** — typed, pure, tested once, reused everywhere; never re-roll the loop.
- Pair structure + algorithm: sorted array ⇄ binary search, adjacency `Map<K,Set>` ⇄ BFS/DFS, `LRUCache` ⇄ memoization.
- Debounce/throttle/memoize UI-driven work; feed debounced values into stable query keys (`07`).
- Hold entity collections **normalized** (`byId` + `allIds`) when you need id-addressing; otherwise let TanStack Query own the list.
- Map/Set/normalized state are mutable — **always write a fresh container** (React/Zustand/Query change detection).
- Under `noUncheckedIndexedAccess`, treat every `Map.get` / index as `T | undefined` — narrow it, never `!`.
- Don't over-engineer: tiny N → plain code; reach for algorithms when N is large or the path is hot (measure vs `14`).
