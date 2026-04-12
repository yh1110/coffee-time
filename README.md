# coffee-time マーケットプレイス

> Claude Code 向けの多角的コードレビュー・プラグイン「**coffee-time**」を配布するマーケットプレイスリポジトリ。

## 収録プラグイン

| プラグイン | 説明 |
| --- | --- |
| [`coffee-time`](./plugins/coffee-time) | 9 観点（可読性・ロジック・セキュリティ・パフォーマンス・エラーハンドリング・テスト・API 設計・データベース・フロントエンド）で多角的にレビューするプラグイン。スラッシュコマンド `code-review` / `review-pr` / `security-review` / `performance-review` と `code-reviewer` サブエージェントを同梱。 |

## インストール

Claude Code のセッション内で次を実行してください。

```text
/plugin marketplace add yh1110/coffee-time
/plugin install coffee-time@coffee-time-marketplace
```

ローカルクローンから使う場合:

```text
/plugin marketplace add /path/to/coffee-time
/plugin install coffee-time@coffee-time-marketplace
```

## 使用例

```text
/coffee-time:code-review                    # 直近の変更をレビュー
/coffee-time:code-review src/api/auth.ts    # ファイル指定
/coffee-time:code-review 1234               # PR 番号指定
/coffee-time:review-pr 1234
/coffee-time:security-review src/
/coffee-time:performance-review src/workers/
```

## 開発者向け: ローカル検証

```bash
git clone https://github.com/yh1110/coffee-time.git
cd coffee-time
# Claude Code から以下を実行して読み込む:
#   /plugin marketplace add .
#   /plugin install coffee-time@coffee-time-marketplace
```

プラグイン構造は公式ドキュメントに準拠しています。

- Plugins reference: https://code.claude.com/docs/en/plugins-reference.md
- Plugin marketplaces: https://code.claude.com/docs/en/plugin-marketplaces.md

## ディレクトリ構造

```
coffee-time/
├── .claude-plugin/
│   └── marketplace.json            # マーケットプレイス定義
├── plugins/
│   └── coffee-time/                # プラグイン本体
│       ├── .claude-plugin/
│       │   └── plugin.json
│       ├── commands/               # スラッシュコマンド（プロンプトは英語）
│       ├── agents/                 # サブエージェント（プロンプトは英語）
│       └── README.md
├── LICENSE
└── README.md
```

## 表記方針

- **プロンプト（commands / agents の md ファイル）は英語**: モデルへの指示精度を優先。
- **最終出力は日本語**: レビュー結果・README・利用者向けドキュメントはすべて日本語。

## ライセンス

MIT
