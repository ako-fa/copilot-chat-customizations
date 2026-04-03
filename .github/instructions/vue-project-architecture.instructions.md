---
description: 'Vue 3 + Nuxt 4 プロジェクトの構造設計ルール'
applyTo: 'app/**/*.vue, app/**/*.ts, app/pages/**/*.vue, app/layouts/**/*.vue'
---

# Vue 3 + Nuxt 4 プロジェクト構造設計ルール

## 基本原則

- コンポーネントをカテゴリ（page / model / ui）で分類し、依存方向を一方向に保つ
- ページの実体は `components/page/` に配置し、`pages/*.vue` はルーティング定義のみとする
- 見た目を伴わない機能は Composable で実装する

## コンポーネント分類

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

## 参考リンク

- [Nuxt 4 公式ドキュメント](https://nuxt.com/docs/4.x/getting-started/introduction)
- [SPA Component の推しディレクトリ構成](https://zenn.dev/knowledgework/articles/99f8047555f700)

## チェックリスト

- コンポーネントが適切なカテゴリ（page / model / ui）に配置されているか
- 依存方向が page → model → ui → composables の一方向を守っているか
- 命名規則（PascalCase、use prefix 等）に準拠しているか
- `pages/*.vue` がルーティング定義のみになっているか
- 自動インポート対象外の `utils/` は明示的にインポートしているか
