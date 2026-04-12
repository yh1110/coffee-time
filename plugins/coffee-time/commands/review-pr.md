---
description: GitHub PR のレビュー（gh CLI 使用、9 観点適用、出力は日本語）
argument-hint: "<PR number | PR URL>"
allowed-tools: Bash, Read, Grep, Glob
---

# Coffee Time — Pull Request Review

Target PR: $ARGUMENTS

This command reuses the 9-axis rubric from `/coffee-time:code-review`, with additional PR-level checks (title / description / scope / migrations).

---

## 1. Gather PR context

```
gh pr view $ARGUMENTS --json title,body,author,baseRefName,headRefName,files,additions,deletions,labels,reviewRequests
gh pr diff $ARGUMENTS
```

If the diff is larger than ~1000 lines, prioritize by risk:
1. Auth / authorization code
2. DB migrations & schema changes
3. API endpoints / server-side handlers
4. Payment / money / PII handling
5. Everything else (summarize)

Before forming opinions, use `Grep` to find callers of changed functions and verify tests were updated alongside the change.

---

## 2. Apply the 9-axis review

Apply every axis (A1–A9) defined in `/coffee-time:code-review`:

- A1 Readability & Maintainability
- A2 Logic & Correctness
- A3 Security
- A4 Performance
- A5 Error Handling
- A6 Tests
- A7 API Design
- A8 Database
- A9 Frontend

Tag each finding with `[A<n>-<axis>]` and cite `file:line`.

---

## 3. PR-specific checks

- **Title & description**: does the PR body explain *why*, not just *what*? Does the title match the actual change?
- **Scope**: is this PR doing one logical thing, or should it be split?
- **Breaking changes**: is there a migration path, rollback plan, backward-compat note?
- **Migrations**: if a schema migration exists, is it reversible? Any risky `NOT NULL` add on a large table?
- **Feature flag / rollout**: gated behind a flag when appropriate?
- **CI status**: mention failing checks if `gh pr checks $ARGUMENTS` reports failures.

---

## 4. Output format (in **Japanese**)

```markdown
## PR サマリ
- タイトル: <title>
- 作成者: <author>
- ブランチ: <head> → <base>
- 変更規模: +<additions> / -<deletions> (<files> ファイル)
- 目的: <1–2 文で PR の狙いを要約>

## 総合判定
<マージ可 / 修正後マージ / 要再設計 のいずれか + 1–2 文の根拠>

## 指摘事項

### 🔴 Critical (マージブロッカー)
- `file.ts:42` [A3-security] — <問題>
  - 理由: ...
  - 提案: ...

### 🟡 Major (修正推奨)
- ...

### 🔵 Minor / Nit
- ...

## 観点別サマリ
- A1 可読性・保守性: ...
- A2 ロジック・正確性: ...
- A3 セキュリティ: ...
- A4 パフォーマンス: ...
- A5 エラーハンドリング: ...
- A6 テスト: ...
- A7 API 設計: ...
- A8 データベース: ...
- A9 フロントエンド: ...

## PR 全体への所見
- 分割すべきか / スコープ / マイグレーション安全性 / 追加で必要なテスト など

## 良かった点
- ...

## 推奨アクション
- [ ] ...
```

---

## 5. Rules

- **Never run `gh pr review` / `gh pr comment` / `gh pr merge`** — output is for the user only. Posting requires explicit instruction.
- Findings without a `file:line` citation MUST be dropped.
- Quote the PR description only when relevant; do not regurgitate it.
- Output must be in Japanese.
