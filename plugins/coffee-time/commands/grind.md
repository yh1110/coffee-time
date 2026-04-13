---
description: パフォーマンス特化レビュー（計算量・I/O・メモリ・FE/BE 固有、出力は日本語）
argument-hint: "[file path | empty(=recent changes)]"
allowed-tools: Read, Grep, Glob, Bash
---

# Coffee Time — Grind

Target: $ARGUMENTS

Findings must be grounded in the code and the **complexity class**, not vibes. Say "O(n²) with n = request count" rather than "feels slow".

---

## 1. Axes

### P1. Algorithmic complexity
- Nested loops (O(n²)+), needless full scans.
- Sort / search algorithm fit.

### P2. Data access
- N+1 queries (DB / HTTP call inside a loop).
- `SELECT *` fetching unused columns; missing JOIN.
- Index-incompatible `WHERE` (function on indexed column, leading `%` in `LIKE`).
- Cacheable data re-fetched each request.

### P3. I/O
- Sync I/O blocking the event loop (Node `readFileSync`, etc.).
- Serial network calls that could be `Promise.all`.
- Unbuffered bulk writes.

### P4. Memory
- Loading entire datasets into memory when stream would do.
- Leak patterns — closure retention, event listener not removed, interval / timeout not cleared, subscription not unsubscribed.
- Unnecessary deep clones.

### P5. Frontend
- Unnecessary re-renders: wrong deps array, inline functions passed to memoized children, missing `key`.
- Large bundle — candidate for dynamic `import()` / code splitting.
- Forced reflow (layout-thrashing reads after writes).
- Image sizing / lazy loading / modern formats (AVIF / WebP).

### P6. Backend
- Worker thread / goroutine / async mis-selection.
- DB connection pool exhaustion risk.
- Redundant serialize / deserialize.

---

## 2. Severity

- 🔴 **High Impact**: will degrade noticeably at realistic scale.
- 🟡 **Medium Impact**: matters under load but not critical today.
- 🔵 **Low / Micro**: observable only in benchmarks; may not be worth the readability cost.

---

## 3. Output format (in **Japanese**)

```markdown
## パフォーマンス評価
<ホットパス / クリティカルパスの特定 + 全体所見>

## 指摘事項

### 🔴 High Impact (計算量・スケール劣化)
- `file:line` [P2-data-access] — <問題> (現在の計算量: O(...))
  - 改善案: <実装例>
  - 期待計算量: O(...)

### 🟡 Medium Impact
### 🔵 Low / Micro

## 計測推奨
- ベンチ / プロファイリングすべき箇所
- 想定負荷条件（同時接続・データ量など）
```

---

## 4. Rules

- Distinguish **micro-optimization** (readability cost > gain) from **real bottlenecks**.
- Quantify: "O(n²) where n grows as …", not "might be slow".
- Flag improvements that rely on untested runtime behavior as "仮説" (hypothesis).
- Output in Japanese.
