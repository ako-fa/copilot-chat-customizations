---
description: 'TypeScript における型安全の徹底と落とし穴回避のためのガイドライン'
applyTo: '**/*.ts, app/**/*.vue, **/*.{spec,test}.{ts,js,vue}'
---

# TypeScript 型安全ガイドライン

TypeScript で型安全性を最大化し、保守性を高めるための指針。

## General Instructions

- `strict: true` を前提に設計し、暗黙の any・推論任せを避ける
- 値の境界（入力・API・外部依存）で必ず型を与え、防御的に狭める
- 公開 API（関数/Composable/コンポーネント Props）はすべて型注釈を付ける
- **`any` は絶対禁止**。`eslint-disable` で回避することも禁止する
- 実行時の失敗を早期に顕在化させる（fail fast）ため、未到達分岐は `never` で網羅性チェックする
- 返り値は必ず記載する。たとえ `void` でも記載する

## any 禁止の徹底

`any` 型は型安全性を完全に破壊するため、**いかなる理由があっても使用禁止**。

### 禁止パターン

- `eslint-disable-next-line @typescript-eslint/no-explicit-any` による許可
- `window as any` でグローバル変数にアクセス
- 型定義を怠って `any` に逃げる

### 正しい対処方法

1. 適切な型定義を作成する（`interface` / `type`）
2. `unknown` で受けて型ガードで絞り込む
3. 外部ライブラリは `import` で型を取得する
4. 型定義がない場合は `declare module` で自分で定義する

### 発見時の対応

- コードレビューで発見したら**即座に拒否**する
- 既存コードで発見したら適切な型定義に置き換えるリファクタリングを行う
- 「時間がない」「後で直す」は理由にならない

## 型の選択と表現

- `unknown` を使って境界を受け、内部で絞り込む
- リテラル型と `as const` を活用し、ユニオン/判別可能ユニオンで状態を表現する
- `enum` より `const enum` またはリテラルユニオン + オブジェクトマップを優先する
- `type` と `interface` は混在させず、プロジェクト方針に沿って統一する

## 関数・API 設計

- パラメータと戻り値に明示的な型を付ける（公開 API では必須）
- `Promise` はジェネリクスで中身の型を指定する
- 可変引数やオプション引数は個別プロパティのオプショナルで表現する
- 例外は例外として扱い、必要なら Result 型で統一する

## オブジェクトと配列

- 配列は `T[]` で統一し、可変操作が多い場合は `readonly T[]` に揃える
- オブジェクト公開時は `Readonly<T>` でラップし、外部からの破壊的変更を防ぐ
- `Record<K, V>` はキーが列挙可能なときのみ使い、動的キーには型ガードを併用する

## スコープと絞り込み

- 可能な限り `const`、再代入が必要な場合のみ `let` を用いる
- ユニオンは型ガード（`in`/`typeof`/`instanceof`/ユーザー定義ガード）で確実に絞り込む
- スイッチ文は判別可能ユニオン + `default: assertNever(x)` で網羅性を検証する

## Best Practices

- モデル/DTO は専用ファイルに分離し、API レスポンス型とドメイン型を分ける
- Zod などのランタイムバリデーションで外部入力を検証し、型と同期する
- 日付や数値計算はプリミティブで渡さず、単位をコメントまたはブランド型で明示する
- コンポーネント Props は必須/デフォルトを明示し、イベント (emits) も型で定義する
- `JSON.parse` など不明型を返す処理の直後で型ガードまたはスキーマバリデーションを行う

## Verification

- 型チェック: `pnpm run typecheck`
- Lint: `pnpm run lint`
- 網羅性チェック: 判別ユニオンで `default` を `assertNever` にし、コンパイルエラーを確認する

## コードパターン参照

- 型安全なエラーハンドリングパターン: `#skill:error-handling`

## 参考リンク

- [TypeScript ガイドライン (Microsoft)](https://github.com/Microsoft/TypeScript/wiki/Coding-guidelines)
- [Google TypeScript ガイドライン](https://google.github.io/styleguide/tsguide.html)
- [TypeScript ESLint](https://typescript-eslint.io/getting-started)
