# coffee-time

> コーヒー片手に、落ち着いて PR・コードを 9 観点でレビューするための Claude Code プラグイン。

**可読性・ロジック・セキュリティ・パフォーマンス・エラーハンドリング・テスト・API 設計・データベース・フロントエンド** の 9 軸で、根拠ベース (`file:line` 引用必須) の多角的レビューを提供します。

## 含まれるもの

### スラッシュコマンド

| コマンド | 用途 |
| --- | --- |
| `/coffee-time:code-review [path\|PR]` | 9 観点での多角的レビュー。引数がなければ直近の変更が対象。 |
| `/coffee-time:review-pr <PR>` | PR 向けレビュー（`gh` CLI 必須）。タイトル・スコープ・マイグレーションも確認。 |
| `/coffee-time:security-review [path]` | OWASP Top 10 に準拠したセキュリティ特化レビュー。 |
| `/coffee-time:performance-review [path]` | 計算量・I/O・メモリ・FE/BE 固有観点でのパフォーマンスレビュー。 |

### サブエージェント

| エージェント | 用途 |
| --- | --- |
| `code-reviewer` | 9 観点ベースの根拠付きレビューを行う専門エージェント。 |

## 9 軸レビューの観点

| 軸 | 内容 |
| --- | --- |
| A1 | 可読性・保守性（命名・責務・マジック値・コメント・ネスト・dead code・型） |
| A2 | ロジック・正確性（エッジケース・境界値・分岐網羅・await・日付） |
| A3 | セキュリティ（XSS / SQLi / CSRF / 認可 / 入力検証 / 機密情報 / CORS / 依存脆弱性 / IDOR） |
| A4 | パフォーマンス（N+1 / 再レンダリング / ページネーション / アセット / バンドル / Index / リーク） |
| A5 | エラーハンドリング（try-catch / ユーザーメッセージ / ログ / API 形式 / タイムアウト / リトライ） |
| A6 | テスト（正常・異常系 / 独立性 / モック適切性 / CI / カバレッジ） |
| A7 | API 設計（REST 準拠 / メソッド / レスポンス形式 / ステータスコード / バージョニング / 型） |
| A8 | データベース（マイグレーション / 正規化 / 制約 / トランザクション / 大量データ） |
| A9 | フロントエンド（レスポンシブ / a11y / ブラウザ互換 / 状態管理 / 責務分離 / フォーム / ローディング） |

## インストール

```text
/plugin marketplace add yh1110/coffee-time
/plugin install coffee-time@coffee-time-marketplace
```

## 使い方

```text
/coffee-time:code-review                    # 直近の変更をレビュー
/coffee-time:code-review src/api/auth.ts    # ファイルを指定
/coffee-time:code-review 1234               # PR 番号を指定
/coffee-time:review-pr 1234
/coffee-time:security-review src/
/coffee-time:performance-review src/workers/
```

サブエージェントは Claude が自動で選ぶか、`code-reviewer エージェントでレビューして` のように指示できます。

## 設計方針

- **根拠ベース**: すべての指摘に `file:line` の引用を必須化し、推測での批判を禁止。
- **優先度を誇張しない**: Lint で拾える程度のものを Critical 扱いにしない。
- **肯定的評価も含む**: 良く書けている箇所も明示的に拾う。
- **サイレントフェイル検出**: `catch` で握りつぶすような実装を重点的に検出。
- **出力は日本語、プロンプトは英語**: モデルに渡す命令は英語で統一し、利用者への最終出力は日本語で統一。
