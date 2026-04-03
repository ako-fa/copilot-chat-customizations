---
description: 'Vue 3 + Nuxt 4 のコンポーネント実装ルール'
applyTo: 'app/**/*.vue, app/composables/**/*.ts'
---

# Vue 3 + Nuxt 4 コンポーネント実装ルール

## 基本原則

- `<script setup lang="ts">` を必ず使用する
- 要素順は **`<script>` → `<template>` → `<style>`** を厳守する
- `<style>` にはデフォルトで `scoped` を付与する

## 型安全

- Props、Emits、Expose は TypeScript で明示的に型定義する
- すべての Props に型・必須/デフォルトを与える
- `any` は絶対禁止（詳細は `typescript-guidelines.instructions.md` を参照）

## リアクティビティ

- 単純な値は `ref` を使用する（`reactive` より推奨）
- `computed` はキャッシュ対象の導出値に使用する
- `watch` は副作用（API 呼び出し等）が必要な監視に使用する
- 外部に公開する ref 値は `readonly()` で保護する

## テンプレート

- `v-for` には **必ず** 安定した `key` を付与する
- 同一要素で `v-if` と `v-for` を併用しない（computed で事前フィルタする）
- コンポーネント名は SFC では PascalCase、DOM テンプレートでは kebab-case
- テンプレート式を複雑にしない（computed/method に移す）
- `v-html` はサニタイズ済みかつ信頼できる文字列に限定する

## Props 設計

- Props は不変（immutable）として扱う。直接変更は禁止
- 変更は `emit` 経由で親に委譲する

## イベント

- イベント名は kebab-case
- 複雑なペイロードは型定義する

## テスト

- 何かを実装したら、**必ず** テストも実装する
- 修正した場合もテストの修正を行う
- テストが未実装のファイルを発見したら直ちにテストを実装する

## コードパターン参照

- コンポーネントパターン: `#skill:vue-component`
- Composable パターン: `#skill:composable`

## ワークフロー参照

- 実装ワークフロー: `#agent:implement`
- リファクタリングワークフロー: `#agent:refactor`

## 参考リンク

- [Vue 3 公式ドキュメント](https://vuejs.org/)
- [Vue 3 Composition API](https://vuejs.org/api/composition-api-setup.html)

## チェックリスト

- `<script setup lang="ts">` を使用しているか
- SFC の要素順（script → template → style）を守っているか
- Props・Emits に TypeScript 型定義があるか
- `v-for` に安定した `key` が付与されているか
- `v-if` と `v-for` が同一要素で併用されていないか
- テストが実装とセットで作成されているか
