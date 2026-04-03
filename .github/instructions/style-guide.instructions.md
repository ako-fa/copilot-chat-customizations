---
description: "Vue3 / Nuxt4 プロジェクトにおける SCSS と Tailwind CSS の役割分担ルール"
applyTo: "**/*.vue, **/*.scss, tailwind.config.ts"
---

# スタイルガイド：SCSS vs Tailwind CSS 役割分担

## 基本原則

- **CSS フレームワーク**: Tailwind CSS v4（Oxide エンジン、Vite プラグイン統合）
- **CSS プリプロセッサ**: SCSS（Dart Sass）
- **アニメーション**: GSAP 3

### 1. スペーシング・レイアウトは Tailwind が優先

- `grid`, `flex`, `gap`, `padding`, `margin` は Tailwind ユーティリティを使用する
- レスポンシブ対応（`md:`, `lg:`）は Tailwind で統一する
- SCSS でレイアウトを制御することは避ける

### 2. 複雑なアニメーションは SCSS または GSAP

- **SCSS**: CSS のみで完結する複雑な `@keyframes` アニメーション
- **GSAP**: ユーザーインタラクションや動的な値に応じるアニメーション

### 3. 計算値・条件分岐は SCSS

- `calc()`, `clamp()` を使った動的な値
- SCSS 変数を使った計算
- 複雑なネストされたセレクタ

### 4. Vue の条件付きクラスは Tailwind が優先

- 単純な状態変更は `:class` で Tailwind ユーティリティを分岐する
- 複数プロパティを一度に変更する複雑な状態のみ SCSS クラスを使う

## 判定フローチャート

```

    ↓
 レイアウト・スペーシング・色？ ─→ YES ───→ 【Tailwind】

 NO
    ↓
 Tailwind ユーティリティで表現できる？ ─→ YES ───→ 【Tailwind】

 NO
    ↓
 @keyframes / 複雑なセレクタ / calc()？ ─→ YES ───→ 【SCSS】

 NO
    ↓
 GSAP で値を動的に制御すべき？ ─→ YES ───→ 初期値を :style で設定
                                           GSAP で値を制御
 NO → JavaScript で制御すべき事項
```

## Tailwind 担当領域

| 用途               | クラス例                                               |
| ------------------ | ------------------------------------------------------ |
| グリッドレイアウト | `grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3`       |
| Flex レイアウト    | `flex flex-col items-center justify-between`           |
| スペーシング       | `p-6 md:p-8`, `gap-4`, `mx-auto`                       |
| タイポグラフィ     | `text-2xl font-bold leading-relaxed text-gray-900`     |
| 背景・ボーダー     | `bg-white border border-gray-300 rounded-lg shadow-lg` |
| ホバー・フォーカス | `hover:bg-blue-600 transition-colors duration-200`     |

## SCSS 担当領域

| 用途                 | 使用場面                               |
| -------------------- | -------------------------------------- |
| `@keyframes`         | 複数ステップのアニメーション           |
| 複雑なセレクタ       | 親状態に基づく子要素のスタイル変更     |
| `calc()` / `clamp()` | 動的に計算すべき値                     |
| SCSS 変数            | 複数箇所で使用する計算値・カラー       |
| scoped スタイル      | Vue コンポーネント固有の複雑なスタイル |
| SCSS ネスト          | 3 階層までを推奨                       |

## GSAP との連携ルール

- アニメーション対象要素の初期値は `:style` で設定する（SCSS/Tailwind ではない）
- GSAP で動かす値は CSS カスタムプロパティ（`--*`）を介して管理してよい
- GSAP 制御要素に CSS `transition` を混在させない
- 状態制御クラス（`is-animating` 等）のみ SCSS で定義可能
- `will-change` はパフォーマンス最適化として SCSS で設定可能

## ファイル構成

```
app/
 assets/
   └── style/
       ├── scss/             # SCSS 変数・ミックスイン
       └── css/              # Tailwind エントリーポイント
```

> **パス注意**: ディレクトリは `app/`（`src/` ではない）

## コードパターン参照

- スタイリングパターン: `#skill:styling`

## ワークフロー参照

- スタイリング実装ワークフロー: `#agent:style`

## 参考リンク

- [Tailwind CSS 公式ドキュメント](https://tailwindcss.com/docs/installation/using-vite)
- [Tailwind CSS v4 リリース](https://tailwindcss.com/blog/tailwindcss-v4)
- [SCSS 公式ドキュメント](https://sass-lang.com/documentation/)
- [Vue3 Style Guide](https://vuejs.org/style-guide/)

## チェックリスト

- Tailwind と SCSS の担当領域が判定フローチャートに従って分離されているか
- レイアウト・スペーシング・レスポンシブ対応に Tailwind を使用しているか
- 複雑なアニメーション・複雑なセレクタ・`calc()` / `clamp()` には SCSS を使用しているか
- GSAP との連携で初期値設定と CSS カスタムプロパティ活用の方針が守られているか
- GSAP 制御要素に CSS `transition` を混在させていないか
- ファイル構成のルール（`app/assets/style/scss` と `app/assets/style/css`）に従っているか
