---
name: refactor
description: "リファクタリングの判断基準・手順・パターン集"
---

# Refactor Skill

既存コードの振る舞いを変えずに、構造だけを改善するための汎用スキル。

## 基本原則

- 実装前に、対象変更でリファクタリングすべき点がないか必ず検討する
- リファクタリングの必要性を見逃すことを許容しない
- 動作変更を禁止し、構造改善に限定する
- 不要な抽象化・不必要な分割・過剰設計を禁止する（YAGNI）

## フェイズ分割ルール

- 変更を安全に分割できる場合は、フェイズ単位で段階的に実施する
- 各フェイズ完了時に、テスト全件グリーンとカバレッジ 100% を必ず満たす
- フェイズ分割自体が目的化することを禁止する
- フェイズを分けない方が明確で安全な場合は 1 フェイズで完了する

## コードスメル検出基準

- 1 つの関数・クラスが複数責務を持つ
- 同一または類似ロジックが 2 箇所以上に重複する
- 意味の不明瞭な命名（略語乱用、役割不一致）
- 引数過多、分岐過多、ネスト過多で読解コストが高い
- ドメイン層が UI 層を参照するなど、依存方向が逆転している
- 副作用と純粋計算が密結合していてテストしづらい
- 使われないコード、死蔵設定、到達不能分岐が残っている

## パターン 1: 抽出（Extract）

責務が混在したコードから、再利用可能な単位を抽出する。

### 例

```typescript
// 抽出前: 関数内に正規化ロジックが埋め込まれている
export function createUser(input: { name: string; email: string }) {
  const normalizedName = input.name.trim();
  const normalizedEmail = input.email.trim().toLowerCase();
  return { name: normalizedName, email: normalizedEmail };
}

// 抽出後: 正規化ロジックを独立関数へ抽出
export function normalizeUser(input: { name: string; email: string }) {
  return {
    name: input.name.trim(),
    email: input.email.trim().toLowerCase(),
  };
}

export function createUser(input: { name: string; email: string }) {
  return normalizeUser(input);
}
```

## パターン 2: 統合（Consolidate）

重複する分岐や計算を 1 か所に集約し、変更点を単一化する。

### 例

```typescript
// 統合前: バリデーションが複数箇所で重複
function validateName(name: string) {
  if (!name || name.length > 50) throw new Error("invalid name");
}

function validateDisplayName(displayName: string) {
  if (!displayName || displayName.length > 50) throw new Error("invalid name");
}

// 統合後: 共通関数に集約
function validateRequiredText(value: string, maxLength: number, field: string) {
  if (!value || value.length > maxLength) {
    throw new Error(`invalid ${field}`);
  }
}
```

## パターン 3: 移動（Move）

責務に対して不適切な配置を修正し、依存方向を正規化する。

### 原則

- 上位レイヤーは下位レイヤーに依存できる
- 下位レイヤーが上位レイヤーへ依存することを禁止する
- 共有契約（型・インターフェース）は中立モジュールへ移動する

## パターン 4: 分割（Split）

肥大化したモジュールを責務ごとに分割する。

### 判断基準

- 1 ファイル内で複数の独立ユースケースを扱っている
- 変更理由の異なる処理が同居している
- テストを分けて書けないため失敗時の原因切り分けが困難

## パターン 5: 不要コード削除（Delete）

未使用コードを削除し、認知負荷と保守コストを下げる。

### 削除対象

- 参照されない関数・クラス・定数
- 実行されない条件分岐
- 使われない設定値・環境変数

## 実施手順

1. 変更対象の現状テストを実行し、グリーンであることを確認する
2. 既存テストの不足箇所を補う（失敗経路と境界値を優先）
3. 最小単位で構造変更を行う
4. 変更直後に対象テストを再実行する
5. 最後に全テストを実行し、回帰がないことを確認する

## チェックリスト

- 振る舞いの変更がないことをテストで確認したか
- 依存方向が正規化されたか
- 重複コードが減少したか
- 変更理由を第三者が説明できる粒度になっているか
- テスト全件グリーンとカバレッジ 100% を満たしているか
