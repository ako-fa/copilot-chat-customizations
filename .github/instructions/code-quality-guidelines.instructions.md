---
description: 'ESLint、Prettier、型チェック、Git フック を統合したコード品質管理ガイド。プロジェクト全体の一貫性と保守性を確保'
applyTo: '**/*.ts, **/*.js, **/*.vue, **/*.json'
---

# コード品質管理ガイドライン

ESLint、Prettier、vue-tsc、Husky、lint-staged を統合し、継続的なコード品質管理を実現する。

## General Instructions

- すべてのコミット前に自動的に linting と formatting が実行される（Husky + lint-staged）
- コード品質は自動チェックに依存し、手動レビューと組み合わせる
- linting エラーは自動修正可能な場合は修正、手動対応が必要な場合は修正内容を明確にする
- Prettier 設定は統一し、エディタ設定と同期させる
- 型チェックは CI/CD パイプラインでも実行され、型安全性を保証する

## ESLint 設定

- 基盤: `@nuxt/eslint-config`
- `vue/multi-word-component-names`: error（例外: `default`, `index`, `error`, `app`）
- `vue/component-name-in-template-casing`: error（PascalCase）
- `@typescript-eslint/no-explicit-any`: warn
- `no-console`: production では warn、development では off

## Prettier 設定

| 設定           | 値     |
| -------------- | ------ |
| printWidth     | 100    |
| tabWidth       | 2      |
| useTabs        | false  |
| semi           | false  |
| singleQuote    | true   |
| trailingComma  | es5    |
| bracketSpacing | true   |
| arrowParens    | always |
| endOfLine      | lf     |

## ESLint 無視規約

- `eslint-disable` は**理由をコメントに明記**した場合のみ許可する
- `eslint-disable-next-line` を使い、影響範囲を最小限にする
- ファイル全体の `eslint-disable` は原則禁止（やむを得ない場合は `enable` で即座に戻す）
- `@typescript-eslint/no-explicit-any` の無視は **絶対禁止**（`typescript-guidelines.instructions.md` 参照）

## コメント規約

### ドキュメントブロック（JSDoc/TSDoc）

- 公開 API（関数/Composable/コンポーネント Props）に必須
- 目的・内容・注意事項を日本語で記述する（What を記述）

### 実装コメント

- 複雑性がわずかでもある場合に追加する
- 実装理由・分岐条件を日本語で記述する（Why を記述）
- 「何をしているか」のコメントは不要（コードから明らか）

## ファイルサイズ

- 1 ファイルに複数の責務を混在させない
- コンポーネントが大きくなったら Composable に抽出する

## Git フック自動化

### pre-commit（Husky + lint-staged）

.editorconfig .env .env.example .env.local .env.staging .git .github .gitignore .husky .nuxt .nuxtrc .output .pnpm-store .vscode README.md app coverage dist doc eslint.config.mts lint-staged.config.mts node_modules nuxt.config.ts package.json pnpm-lock.yaml pnpm-workspace.yaml prettier.config.ts public stylelint.config.mjs tsconfig.json tsconfig.tsbuildinfo vanta.d.ts vitest.config.mts :-abranch

1. ESLint による自動修正
2. Prettier による自動フォーマット
3. vue-tsc による型チェック

### pre-push

push 前に全プロジェクト対象の型チェック: `vue-tsc --noEmit`

## Verification

```bash
# ESLint チェック
pnpm run lint

# 型チェック
pnpm run typecheck

# フォーマット確認
pnpm run format

# テスト実行
pnpm run test
```

## よくあるエラーと対処

| エラー              | 原因                            | 対処法                                       |
| ------------------- | ------------------------------- | -------------------------------------------- |
| `Prettier mismatch` | ESLint と Prettier 設定の不一致 | `pnpm run format` を実行                     |
| `Missing type`      | TypeScript 型定義不足           | `pnpm run typecheck` で確認                  |
| `Rule error`        | ESLint ルール違反               | `pnpm run lint --fix` で修正、または手動対応 |
| `Vue syntax error`  | テンプレート構文エラー          | vue-tsc の警告を確認                         |

## ワークフロー参照

- コードレビューワークフロー: `#agent:review`
- デバッグワークフロー: `#agent:debug`

## 参考リンク

- [ESLint 公式ドキュメント](https://eslint.org/)
- [Prettier 公式ドキュメント](https://prettier.io/)
- [TypeScript 公式ドキュメント](https://www.typescriptlang.org/)
- [Husky GitHub](https://typicode.github.io/husky/)
- [lint-staged GitHub](https://github.com/lint-staged/lint-staged)
