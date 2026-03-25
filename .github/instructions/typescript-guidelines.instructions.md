---
applyTo: "**/*.{ts,tsx}"
description: "TypeScript コーディング規約"
---

# TypeScript コーディング規約

TypeScript の型安全性と保守性を高水準で維持するための汎用ガイドライン。

## 基本原則

- `strict: true` を前提とし、`strictNullChecks` を無効化しない
- **`any` 型の使用を禁止**する
- 境界入力（HTTP、ファイル、環境変数、外部 API など）は `unknown` で受けて検証する
- 公開 API（関数、クラス、モジュールのエクスポート）には引数型と戻り値型を明示する
- 未到達分岐は `never` を使って網羅性を強制する

## any 禁止と unknown 活用

- `any` を回避するために `unknown` を使い、型ガードで安全に絞り込む
- 「一時的な回避」を理由とした `eslint-disable` は認めない
- 型定義が不足している場合は、`type` / `interface` / `declare module` で補完する

## ジェネリクス活用

- 再利用可能な処理はジェネリクスで抽象化し、呼び出し側に型情報を伝播させる
- `Promise<T>`、`Map<K, V>`、`Record<K, V>` などは型引数を省略しない
- 汎用関数は制約付きジェネリクス（`<T extends ...>`）を優先し、過度な型アサーションを避ける

## 型ガード実装パターン

- `typeof`、`instanceof`、`in` を用いて分岐ごとに型を狭める
- 複雑な判定はユーザー定義型ガード（`value is X`）として関数化する
- 不正値を受けた場合は早期に例外またはエラー値を返し、fail fast を徹底する

## 型設計と命名規則

- 型名、インターフェース名、列挙的な型名は `PascalCase` を使用する
- 型引数は意味のある名称を優先し、単文字は局所的文脈でのみ許可する
- 判別可能ユニオンの判別キーは明確な固定名（例: `kind`, `type`）で統一する

## 検証

- 型チェックを CI に組み込み、エラーが 1 件でもある状態でのマージを禁止する
- Lint で `no-explicit-any`、`no-unsafe-*` 系ルールを有効化する

## 参考リンク

- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [TypeScript ESLint](https://typescript-eslint.io/)
