---
description: セキュリティ特化レビュー（OWASP Top 10 準拠、出力は日本語）
argument-hint: "[file path | empty(=recent changes)]"
allowed-tools: Read, Grep, Glob, Bash
---

# Coffee Time — Security Review

Target: $ARGUMENTS

Security-focused pass. When the target is broad, prioritize:
1. Code that touches user input (forms, query params, request bodies).
2. Auth / authorization code.
3. External I/O (HTTP fetch, file system, subprocess).
4. Database access.

---

## 1. OWASP Top 10 checklist

### A01 Broken Access Control
- Missing authZ checks; IDOR (operating on another user's resource).
- Admin-only endpoints not gated.

### A02 Cryptographic Failures
- Secrets stored in plaintext; weak hashes (MD5 / SHA1); fixed IV.
- HTTPS enforced; Cookie `Secure` / `HttpOnly` / `SameSite` set.

### A03 Injection
- SQL: raw string concatenation vs prepared statements / parameter binding.
- Command injection: `exec`, `child_process`, `shell=True`.
- LDAP / XPath / template injection.
- XSS: unescaped DOM insertion; `dangerouslySetInnerHTML` without sanitizer.

### A04 Insecure Design
- Rate limiting on abusable endpoints.
- CSRF protection for state-changing requests.
- Business-logic authorization (can user X modify resource Y?).

### A05 Security Misconfiguration
- Default passwords; debug mode in production; directory listing.
- Over-permissive CORS (`*` with `Access-Control-Allow-Credentials: true`).

### A06 Vulnerable & Outdated Components
- Stale dependencies; known-vulnerable library usage patterns.

### A07 Identification & Authentication Failures
- Weak password policy, session fixation, token leakage paths.
- JWT: `alg: none`, secret handling, expiration.

### A08 Software & Data Integrity Failures
- Unsigned deserialization (`pickle`, `yaml.load`, Java serialization).
- Unverified third-party CDN / script.

### A09 Logging & Monitoring Failures
- Auth failures / authZ violations not logged.
- **Secrets / tokens / PII printed to logs**.

### A10 Server-Side Request Forgery
- User-controlled URLs used in HTTP fetch; no allowlist; metadata-service access not blocked.

---

## 2. Severity

- 🔴 **Critical**: exploitable now, data exposure / auth bypass.
- 🟡 **High**: exploitable with conditions (specific role, specific input).
- 🔵 **Medium**: hardening gap; not directly exploitable.
- ⚪ **Low / Informational**: defense-in-depth suggestion.

---

## 3. Output format (in **Japanese**)

```markdown
## セキュリティ評価
<全体リスク: Low / Medium / High / Critical と 1–2 文の根拠>

## 指摘事項

### 🔴 Critical (即時対応)
- `file:line` [カテゴリ A03-injection] — <脆弱性の要点>
  - 影響: ...
  - 再現条件: <概念レベル、動作する攻撃ペイロードは書かない>
  - 修正案: <コード例>

### 🟡 High
### 🔵 Medium
### ⚪ Low / Informational

## 推奨事項
- 防御的プログラミング・設計上の改善
```

---

## 4. Rules

- **Do not fabricate vulnerabilities.** False positives erode trust.
- Explain exploitation **conceptually only**. Do not write working payloads.
- Every finding needs a `file:line` citation.
- Output in Japanese.
