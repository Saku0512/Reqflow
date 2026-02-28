# Reqflow

> 要件定義書の作成からタスク管理・GitHub連携まで、PdMのワークフローを一気通貫で支えるデスクトップアプリ

---

## 概要・コンセプト

既存のプロジェクト管理ツールは「タスク管理」と「ドキュメント管理」が分断されており、PdMが要件定義書を書いてからJiraやGitHub Projectsにタスクを起こすまでの作業は手動・二重管理になりがちです。

**Reqflow** はこの問題を解決します。

- 要件定義書をIPAのフレームワークに沿って作成・管理できる
- 書いた要件からそのままGitHub IssueおよびProjectsのタスクを生成できる
- PdMがエンジニアへ指示する際の「要件とタスクのトレーサビリティ」が常に保たれる

```
要件定義書 → 機能一覧 → GitHub Issue生成 → 進捗管理
```

このフローがひとつのデスクトップアプリで完結します。

---

## 機能一覧

### ✅ MVP（実装済み予定）

- プロジェクト作成・管理
- 要件定義書エディタ（Markdown形式）
  - 業務要件 / 機能要件 / 非機能要件 / 制約条件のセクション管理
  - 未決定事項（Open Issues）のインライン管理
  - セクションごとのステータス管理（Draft / Review / Approved）
- 自動保存

### 🚧 今後実装予定

- 要件 → GitHub Issue 自動生成
- GitHub Projects との双方向同期
- タスクボード（カンバン / リスト）
- 担当者アサイン・進捗管理
- AI による要件定義書ドラフト生成（Anthropic API）

---

## 技術スタック

| レイヤー | 技術 |
|----------|------|
| デスクトップフレームワーク | [Wails v2](https://wails.io/) |
| フロントエンド | [Svelte](https://svelte.dev/) |
| バックエンド | [Go](https://golang.org/) |
| データベース | SQLite |
| 外部連携 | GitHub GraphQL API v4 |

### アーキテクチャの特徴

WailsによりGoのメソッドをフロントエンドから直接呼び出せます。HTTPサーバーを別途立てる必要がなく、シンプルな構成で動作します。

```
Svelte (UI)
    ↕ Wails Bind（GoのStructメソッドを直接呼び出し）
Go (ビジネスロジック・DB・GitHub API)
    ↕
SQLite / GitHub GraphQL API
```

---

## アーキテクチャ図

```
reqflow/
├── frontend/               # Svelte
│   └── src/
│       ├── lib/
│       │   ├── components/ # UIコンポーネント
│       │   └── wailsjs/    # 自動生成されるGoバインディング
│       └── routes/
│           ├── +page.svelte          # ダッシュボード
│           └── projects/[id]/
│               ├── requirements/     # 要件定義書エディタ
│               └── tasks/           # タスクボード
│
├── app.go                  # Wailsアプリ本体・Goメソッド定義
├── model/                  # データモデル
├── repository/             # DB操作
├── handler/                # GitHub API連携
├── db/
│   └── schema.sql
└── main.go
```

---

## セットアップ手順

### 必要な環境

- [Go](https://golang.org/) 1.21 以上
- [Node.js](https://nodejs.org/) 18 以上
- [Wails CLI](https://wails.io/docs/gettingstarted/installation)

```bash
go install github.com/wailsapp/wails/v2/cmd/wails@latest
```

### インストール

```bash
# リポジトリをクローン
git clone https://github.com/your-username/reqflow.git
cd reqflow

# 依存関係のインストール
go mod tidy
cd frontend && npm install && cd ..
```

### 開発サーバーの起動

```bash
wails dev
```

### ビルド

```bash
wails build
```

ビルド成果物は `build/bin/` に生成されます。

---

## GitHub連携の設定

1. [GitHub Personal Access Token](https://github.com/settings/tokens) を作成
   - 必要なスコープ: `repo`, `project`
2. アプリの設定画面からトークンを入力

---

## 今後のロードマップ

### v0.1 - MVP
- [ ] プロジェクト設計・DB設計
- [ ] 要件定義書エディタ（基本機能）
- [ ] プロジェクト作成・一覧表示

### v0.2 - GitHub連携
- [ ] GitHub認証（PAT）
- [ ] 要件 → GitHub Issue 自動生成
- [ ] GitHub Projects との同期

### v0.3 - タスク管理
- [ ] タスクボード（カンバン）
- [ ] 担当者アサイン
- [ ] 進捗ダッシュボード

### v1.0 - AI機能
- [ ] Anthropic APIによる要件定義書ドラフト自動生成
- [ ] 未決定事項の自動検出・サジェスト

---

## ライセンス

MIT License
