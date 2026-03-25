---
description: "Vue 3 と Nuxt 4 におけるコンポーネント設計とプロジェクト構成の宣言的ルール"
applyTo: "app/**/*.vue, app/**/*.ts, app/composables/**/*.ts, app/pages/**/*.vue, app/layouts/**/*.vue"
---

# Vue 3 + Nuxt 4 ベストプラクティス

## 必須ルール

### SFC（単一ファイルコンポーネント）

- `<script setup lang="ts">` を必ず使用する
- 要素順は **`<script>` → `<template>` → `<style>`** を厳守する
- `<style>` にはデフォルトで `scoped` を付与する

### 型安全

- Props、Emits、Expose は TypeScript で明示的に型定義する
- すべての Props に型・必須/デフォルトを与える
- `any` は絶対禁止（詳細は `typescript-guidelines.instructions.md` を参照）

### リアクティビティ

- 単純な値は `ref` を使用する（`reactive` より推奨）
- `computed` はキャッシュ対象の導出値に使用する
- `watch` は副作用（API 呼び出し等）が必要な監視に使用する
- 外部に公開する ref 値は `readonly()` で保護する

### テンプレート

- `v-for` には **必ず** 安定した `key` を付与する
- 同一要素で `v-if` と `v-for` を併用しない（computed で事前フィルタする）
- コンポーネント名は SFC では PascalCase、DOM テンプレートでは kebab-case
- テンプレート式を複雑にしない（computed/method に移す）
- `v-html` はサニタイズ済みかつ信頼できる文字列に限定する

### Props 設計

- Props は不変（immutable）として扱う。直接変更は禁止
- 変更は `emit` 経由で親に委譲する

### イベント

- イベント名は kebab-case
- 複雑なペイロードは型定義する

### テスト

- 何かを実装したら、**必ず** テストも実装する
- 修正した場合もテストの修正を行う
- テストが未実装のファイルを発見したら直ちにテストを実装する

## コンポーネント分類ルール

### カテゴリ

| カテゴリ   | 定義                                           | ディレクトリ        |
| ---------- | ---------------------------------------------- | ------------------- |
| `page/`    | 1 つのページを表すコンポーネント               | `components/page/`  |
| `model/`   | 特定のドメインモデルに関心を持つコンポーネント | `components/model/` |
| `ui/`      | モデルに関心を持たない汎用 UI コンポーネント   | `components/ui/`    |
| functional | 見た目を伴わない機能 → **Composable で実装**   | `composables/`      |

### page コンポーネント

- `*.page.vue`（レイアウト担当）と `*.vue`（非同期処理担当）を分離する
- `pages/*.vue` はルーティング定義のみ。ページの実体は `components/page/` に配置する
- `Suspense` で非同期コンテンツをラップする

### model コンポーネント

- モデル名をディレクトリで分類する（`user/`, `article/` 等）
- コンポーネント名にモデル名を Prefix として付与する（`UserAvatar`, `ArticleCard`）
- モデルが不要になるとコンポーネントも不要になる関係

### ui コンポーネント

- 複数ページで再利用可能、ドメインロジックに依存しない
- `Base*`（基本 UI 要素）、`App*`（アプリ単位の単一例）、`The*`（ページ固有親レベル）

## 依存ルール

```
page → model → ui → composables（一方向のみ）
```

- 上位から下位への依存は OK
- 下位から上位への依存は NG（例: ui が page を依存してはいけない）
- 同一カテゴリ内の相互参照は OK
- page は page を依存しない（ページは独立）

## 命名規則

| 要素                | 規則                     | 例                                 |
| ------------------- | ------------------------ | ---------------------------------- |
| コンポーネント      | PascalCase、マルチワード | `BaseButton.vue`, `UserAvatar.vue` |
| ページラッパー      | PascalCase + `.page.vue` | `TopPageWrapper.page.vue`          |
| Composable          | camelCase + `use` prefix | `useCounter.ts`, `useKeyBind.ts`   |
| ユーティリティ      | camelCase                | `formatDate.ts`, `helpers.ts`      |
| 定数                | SCREAMING_SNAKE_CASE     | `API_ENDPOINTS`, `MAX_RETRY_COUNT` |
| 型/インターフェース | PascalCase               | `UserProfile`, `ApiResponse`       |

> **Vue スタイルガイド遵守**: コンポーネント名は **必ず** マルチワード。例外は Nuxt 規約上必要な `default`, `index`, `error` のみ。

## 自動インポート

Nuxt は以下を自動インポートする：

- `components/` 配下のすべてのコンポーネント
- `composables/` 配下のすべての Composable
- `utils/` は自動インポート **されない**（明示的にインポートする）

## ディレクトリ構成

```
app/
 components/
   ├── page/              # ページコンポーネント
   ├── model/             # ドメインモデル固有
   └── ui/                # 汎用 UI
 pages/                 # ルーティング定義のみ
 layouts/               # ページレイアウト
 composables/           # 再利用可能なロジック + functional 機能
 plugins/               # Nuxt プラグイン
 utils/                 # ユーティリティ関数
 assets/                # 静的アセット
 types/                 # TypeScript 型定義
```

## コードパターン参照

- コンポーネントパターン: `#skill:vue-component`
- Composable パターン: `#skill:composable`

## ワークフロー参照

- 実装ワークフロー: `#agent:implement`
- リファクタリングワークフロー: `#agent:refactor`

## 参考リンク

- [Vue 3 公式ドキュメント](https://vuejs.org/)
- [Nuxt 4 公式ドキュメント](https://nuxt.com/docs/4.x/getting-started/introduction)
- [Vue 3 Composition API](https://vuejs.org/api/composition-api-setup.html)
- [SPA Component の推しディレクトリ構成](https://zenn.dev/knowledgework/articles/99f8047555f700)
