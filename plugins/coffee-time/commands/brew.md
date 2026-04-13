---
description: コードや PR に対して 9 観点で多角的なコードレビューを実施（出力は日本語）
argument-hint: "[file path | PR number | empty(=recent changes)]"
allowed-tools: Read, Grep, Glob, Bash
---

# Coffee Time — Brew

Target: $ARGUMENTS

Act as a calm, experienced reviewer sitting down with a coffee. Give **evidence-based, non-speculative feedback**. Every finding MUST cite an exact `file:line`.

---

## 1. Scope resolution

Pick the target in this order:

1. If `$ARGUMENTS` looks like a PR number or PR URL, run:
   ```
   gh pr view <id> --json title,body,author,baseRefName,headRefName,files,additions,deletions
   gh pr diff <id>
   ```
2. If `$ARGUMENTS` is a file or directory path, review that path.
3. If `$ARGUMENTS` is empty:
   - Run `git status` and `git diff` — review staged + unstaged changes.
   - If nothing is pending, review the latest commit (`git show HEAD`).
4. For very large diffs (>1000 lines), prioritize files by risk (auth, data access, business-critical paths) and summarize the rest.

Always open the actual changed files with `Read`. Use `Grep` to locate callers and tests for changed symbols.

---

## 2. Review axes (all 9 must be considered)

Tag each finding with the axis, e.g. `[A3-security]`. If an axis has no issues, write "特になし" in the per-axis summary.

### A1. Readability & Maintainability
- Are variable, function, and class names intention-revealing?
- Is each function / method split at the right granularity (single responsibility)?
- Are magic numbers / magic strings extracted into named constants?
- Are comments present where they actually help (not too many, not too few)?
- Is nesting shallow enough? Can early return flatten it?
- Any dead code (unused functions, variables, imports) left behind?
- Does the code follow the project's coding convention / Lint rules?
- Are type definitions appropriate (no `any` abuse)?

### A2. Logic & Correctness
- Does the implementation correctly meet the stated requirement / spec?
- Are edge cases handled (empty array, `null` / `undefined`, `0`, empty string, etc.)?
- Are boundary values handled correctly?
- Any missing branches (`if`-`else` with no `else`; `switch` with no `default`)?
- Are loop termination conditions correct (no accidental infinite loop)?
- No missing `await`; are Promise chains handled correctly?
- Are dates / timezones handled correctly?

### A3. Security
- **XSS**: is user input sanitized? `dangerouslySetInnerHTML` (or equivalent raw-HTML injection) avoided unless clearly safe?
- **SQL injection**: are prepared statements / parameterized bindings used (never string-concatenated SQL)?
- **CSRF**: is token validation implemented for state-changing requests?
- **AuthN / AuthZ**: are permission checks in place (including on API endpoints)?
- **Input validation**: is validation enforced on the **server side** (not only the client)?
- **Secrets**: no passwords / API keys / tokens leaking into logs or responses?
- **CORS**: is the origin allowlist appropriate?
- **Dependencies**: any known-vulnerable packages?
- **IDOR**: is the design such that a user cannot operate on resources they do not own?

### A4. Performance
- Any N+1 queries?
- Unnecessary re-renders? (React: appropriate use — and **non-overuse** — of `useMemo`, `useCallback`, `React.memo`)
- Pagination / infinite scroll considered for large datasets?
- Image / asset optimization (lazy loading, appropriate formats)?
- Are API-call counts optimal (batching, caching)?
- Bundle-size impact — unused library imports, tree-shaking friendliness?
- Are DB indexes set up appropriately?
- Any memory-leak risk (event listeners not removed, timers not cleared)?

### A5. Error Handling
- Are errors caught appropriately with try-catch?
- Are user-facing messages understandable when errors occur?
- Are error logs recorded with enough context (stack trace, request context)?
- Do API error responses follow the spec (status code, error message format)?
- Are network errors / timeouts handled?
- Where appropriate, are retry / fallback strategies implemented?

### A6. Tests
- Are tests added / updated for the new or changed code?
- Are both happy path and error paths covered?
- Are tests independent of each other (no execution-order dependency)?
- Are mocks / stubs used appropriately (not so mock-heavy that the test becomes meaningless)?
- Is CI green?
- Has test coverage not regressed significantly?

### A7. API Design
- Does the endpoint design follow REST (or the project's chosen style)?
- Is the HTTP method appropriate (GET / POST / PUT / DELETE)?
- Is the response shape consistent across endpoints?
- Are status codes correct?
- Is API versioning considered where relevant?
- Are request / response types clearly defined?

### A8. Database
- Is the migration file correct (and rollback-capable)?
- Is the schema normalized (or intentionally denormalized with a reason)?
- Are FK / NOT NULL (and other) constraints set appropriately?
- Are transactions used where multi-statement atomicity is required?
- Is bulk-data insert / update performance considered?

### A9. Frontend
- Is responsive design considered?
- Is accessibility (a11y) considered (`alt`, `aria-label`, keyboard navigation)?
- Any browser-compatibility concerns?
- Is state management appropriate (local state vs global state)?
- Are component responsibilities separated (e.g. Container / Presentational)?
- Is form validation handled appropriately (realtime vs on-submit)?
- Are loading / empty states considered in the UI?

---

## 3. Severity rubric

- **🔴 Critical (ブロッカー)** — data loss, security hole, production break, auth bypass, silent failure that damages user-visible behavior.
- **🟡 Major (修正推奨)** — clear bug in a non-critical path, missing validation, significant perf regression, missing test for new logic.
- **🔵 Minor / Nit** — style suggestion, naming tweak, micro-optimization, documentation gap.

Do NOT inflate severity. Lint-fixable issues are Minor, not Critical.

---

## 4. Output format (must be in **Japanese**)

Produce exactly this structure in Japanese. Every bullet must include a concrete `file:line`.

```markdown
## 概要
<全体評価 1–3 文 / マージ可否の感触>
<PR レビューの場合は: タイトル / 作成者 / 変更規模 +X / -Y (Z files) も追記>

## 指摘事項

### 🔴 Critical (マージブロッカー)
- `path/to/file.ts:42` [A3-security] — <問題の要点>
  - 理由: <なぜ問題か>
  - 提案: <具体的な修正案 / 必要ならコードスニペット>

### 🟡 Major (修正推奨)
- `path/to/file.ts:80` [A2-logic] — ...

### 🔵 Minor / Nit
- `path/to/file.ts:120` [A1-readability] — ...

## 観点別サマリ
- A1 可読性・保守性: <特になし | 指摘 N 件>
- A2 ロジック・正確性: ...
- A3 セキュリティ: ...
- A4 パフォーマンス: ...
- A5 エラーハンドリング: ...
- A6 テスト: ...
- A7 API 設計: ...
- A8 データベース: ...
- A9 フロントエンド: ...

## 良かった点
- <意図的に良く書かれている箇所を具体的に>

## 推奨アクション
- [ ] <具体的 TODO>
```

---

## 5. Rules

- **No hallucinated findings.** If you cannot cite `file:line`, do not include the bullet.
- **No auto-commenting on GitHub.** Do NOT run `gh pr review`, `gh pr comment`, or any write operation. Output stays in chat unless the user explicitly asks to post.
- **No speculative criticism.** Mark a concern as "推測" explicitly if it depends on runtime behavior you cannot verify from the code.
- **Praise where deserved** — include at least one genuine positive observation when the code warrants it.
- **Respect the author.** Review the code, not the person. Tone: 丁寧で建設的.
