# DOC-30 ドメイン定義書

| 項目 | 内容 |
|------|------|
| 書類ID | DOC-30 |
| IPA分類 | DD.10.2 |
| プロジェクト名 | Reqflow |
| 作成日 | 2026-03-01 |
| 作成者 | Saku0512 |
| ステータス | Draft |

---

## 1. 概要

本ドキュメントでは、Reqflowのデータベース（SQLite）およびGo側のバックエンドモデルにおいて利用する、データ項目の属性（データ型、値域、文字長、バリデーションルール）を共通化するためのドメイン（データドメイン）を定義します。これにより、テーブル設計やフロントエンド/バックエンドでの入力チェックの基準を統一します。

## 2. ドメイン定義一覧

### 2.1 識別子（ID）系ドメイン

すべてのレコードの一意性を保証する主キー等に使用するドメインです。

| ドメイン名 | 物理データ型 (SQLite) | Go型 | フォーマット/制約 | 概要・用途 |
| :--- | :--- | :--- | :--- | :--- |
| `ID_UUID` | `TEXT` | `string` (`uuid.UUID`) | 36文字固定、UUIDv4形式 | 主キー（`id`）、外部キー等に使用。分散環境でも一意。 |
| `ExtID_GitHub` | `TEXT` | `string` | 文字列（英数字・ハイフン等） | GitHub IssueのID（GraphQLのNode ID等）やRepository名など、外部システムが発行したIDを格納。 |

### 2.2 テキスト・文字列系ドメイン

名称や説明文などを格納するドメインです。

| ドメイン名 | 物理データ型 (SQLite) | Go型 | フォーマット/制約 | 概要・用途 |
| :--- | :--- | :--- | :--- | :--- |
| `Name_Short` | `TEXT` | `string` | 最大 100文字、NOT NULL、空文字不可 | プロジェクト名（`projects.name`）、タスクタイトル等。 |
| `Name_Medium`| `TEXT` | `string` | 最大 255文字、NOT NULL、空文字不可 | セクションタイトル（`requirement_sections.title`）等。 |
| `Description`| `TEXT` | `string` | 最大長制限なし（運用上は1000文字程度を想定） | プロジェクト説明、未決定事項の内容記述等。 |
| `Content_MD` | `TEXT` | `string` | 最大長制限なし（SQLiteのTEXT制限に準拠）。Markdown形式を期待。 | 要件セクションの本文（`requirement_sections.content`）。 |

### 2.3 区分値・ステータス（Enum）系ドメイン

状態を表すために定義された固定の文字列（マジックストリングの排除用）です。
※Go側では、`type Status string` としてConstで定義しバリデーションを行います。

| ドメイン名 | 物理データ型 (SQLite) | Go型 | 許容値（値域） | 概要・用途 |
| :--- | :--- | :--- | :--- | :--- |
| `Status_Project` | `TEXT` | `string` | `draft`, `active`, `archived` | プロジェクト自体の進行状態。デフォルトは `draft`。 |
| `Status_Section` | `TEXT` | `string` | `draft`, `review`, `approved` | 要件セクションの承認ステータス。Issue生成は `approved` のみ実施可。 |
| `Status_Task` | `TEXT` | `string` | `todo`, `in_progress`, `done` | カンバンボードで管理するタスクステータス。GitHubと同期。 |
| `Section_Type` | `TEXT` | `string` | `business`, `functional`, `non_functional`, `constraint`, `other` | 要件の分類。これによりUIでの表示グループなどを制御する。 |

### 2.4 日付・時刻系ドメイン

| ドメイン名 | 物理データ型 (SQLite) | Go型 | フォーマット/制約 | 概要・用途 |
| :--- | :--- | :--- | :--- | :--- |
| `Timestamp` | `DATETIME` | `time.Time` | ISO8601フォーマット、UTC（またはローカルタイムに統一） | 作成日時（`created_at`）、更新日時（`updated_at`）。トリガー等で自動打刻。 |
| `Date_Due` | `DATE` | `time.Time` | YYYY-MM-DD フォーマット。必要に応じてNULL許容。 | 未決定事項の期限、タスクの期限等。時間情報は持たない。 |

### 2.5 数値・真偽値系ドメイン

| ドメイン名 | 物理データ型 (SQLite) | Go型 | フォーマット/制約 | 概要・用途 |
| :--- | :--- | :--- | :--- | :--- |
| `Flag` | `BOOLEAN` | `bool` | `0` (False) もしくは `1` (True)。デフォルトは `0`。 | 未決定事項の解決済みフラグ（`resolved`）等。 |
| `Order_Index`| `INTEGER` | `int` | 0以上。NULL不可。 | セクションの並び順（ドラッグ＆ドロップによる順序入れ替え）管理等に使用。 |

## 3. バリデーション実装方針

本ドメイン定義に基づく入力バリデーションは、以下の各層で実装します。

1.  **フロントエンド (Svelte):**
    入力フォームにおける文字数制限、必須チェック、およびEnum値に基づくプルダウン選択を提供し、即座なエラーフィードバックを行います。
2.  **バックエンド (Go):**
    APIエンドポイント（Wailsのバインディング関数）内で、構造体タグ（例: `validate:"required,max=100"` など、validatorパッケージの利用）またはカスタムバリデーションロジックにより、不正データのDB流入を最終ブロックします。
3.  **データベース (SQLite):**
    極力 `NOT NULL` 制約や、`CHECK` 制約をスキーマに定義し、データの堅牢性を担保します。（例： `CHECK (status IN ('draft', 'review', 'approved'))`）
