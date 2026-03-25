---
description: "エラーハンドリングの設計パターンと実装テンプレート"
---

# Error Handling Skill

TypeScript 5.9 における型安全なエラーハンドリングの再利用可能なコードパターン。

## カスタムエラークラス

```typescript
/**
 * バリデーションエラー
 *
 * @param field - エラーが発生したフィールド名
 * @param message - エラーメッセージ
 */
class ValidationError extends Error {
  constructor(
    public field: string,
    message: string,
  ) {
    super(message);
    this.name = "ValidationError";
  }
}

// 使用例
try {
  validateEmail(email);
} catch (error) {
  if (error instanceof ValidationError) {
    console.error(`バリデーション失敗: ${error.field}: ${error.message}`);
  } else {
    console.error("不明なエラー", error);
  }
}
```

## エラー分類

運用しやすいように、エラーは次の区分で扱う。

- ネットワークエラー: タイムアウト、接続失敗、一時的な通信断
- バリデーションエラー: 入力値不正、必須項目不足、形式不正
- 認証・認可エラー: 未ログイン、権限不足、期限切れトークン
- システムエラー: 想定外例外、依存サービス障害、内部処理失敗

## 非同期エラーハンドリング

```typescript
/**
 * API からデータを取得する
 *
 * @returns 取得したデータ
 * @throws Error HTTP ステータスが異常な場合
 */
async function fetchData<T>(url: string): Promise<T> {
  const response = await fetch(url);

  if (!response.ok) {
    throw new Error(`HTTP ${response.status}: ${response.statusText}`);
  }

  return (await response.json()) as T;
}
```

## エラーハンドリング戦略

- リトライ: 一時的障害（例: ネットワーク）には上限回数付きで再試行する
- フォールバック: 取得失敗時はキャッシュや既定値へ切り替える
- ユーザー通知: 利用者が次に取るべき行動が分かる文言で通知する

```typescript
async function withRetry<T>(
  task: () => Promise<T>,
  maxRetries = 3,
): Promise<T> {
  let lastError: unknown;

  for (let i = 0; i < maxRetries; i++) {
    try {
      return await task();
    } catch (error) {
      lastError = error;
    }
  }

  throw lastError;
}
```

## エラーレスポンスと表示

フレームワーク固有 API に依存せず、アプリケーション共通の構造で扱う。

```typescript
type ErrorResponse = {
  code: string;
  message: string;
  details?: unknown;
};

function createErrorResponse(
  code: string,
  message: string,
  details?: unknown,
): ErrorResponse {
  return { code, message, details };
}
```

UI では「専用のエラー表示領域」を用意し、処理失敗時にフォールバック表示へ切り替える。

## エラーログ方針

- 利用者向け文言と運用者向け詳細ログを分離する
- ログには発生時刻、エラー種別、再現に必要な最小情報を残す
- パスワード、トークン、個人情報などの機密値をログに出力しない

## 判別可能ユニオンによる状態管理

```typescript
/**
 * 非同期処理の状態を表す判別可能ユニオン型
 *
 * status プロパティで状態を判別し、型の絞り込みを行う
 */
type LoadState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; message: string };

/**
 * ありえない分岐に到達した場合にコンパイルエラーを発生させる
 * switch 文の default で使用して網羅性を保証する
 */
function assertNever(x: never): never {
  throw new Error(`到達不能: ${x}`);
}

function render(state: LoadState<User[]>): string {
  switch (state.status) {
    case "idle":
      return "待機中";
    case "loading":
      return "読み込み中";
    case "success":
      return `${state.data.length} 件`;
    case "error":
      return state.message;
    default:
      assertNever(state); // 未到達分岐の網羅性チェック
  }
}
```

## unknown 型による安全な型ガード

```typescript
/**
 * 不明な入力を安全に文字列に変換する
 *
 * unknown で受けて型ガードで絞り込み、any を使わない
 */
function processData(input: unknown): string {
  if (typeof input === "string") {
    return input;
  }
  if (typeof input === "object" && input !== null && "message" in input) {
    return String((input as { message: unknown }).message);
  }
  throw new Error("無効な入力型");
}
```

## any 禁止 — 正しい対処方法

```typescript
// ❌ 絶対禁止: any 型
// let instance: any = null
// const data: any = someValue
// eslint-disable-next-line @typescript-eslint/no-explicit-any

// ✅ 正しい方法 1: 適切な型定義を作成する
interface PluginInstance {
  init: () => void;
  destroy: () => void;
}
let instance: PluginInstance | null = null;

// ✅ 正しい方法 2: unknown で受けて型ガードで絞り込む
function safeProcess(input: unknown): string {
  if (typeof input === "string") return input;
  throw new Error("Invalid input type");
}

// ✅ 正しい方法 3: 外部ライブラリは直接 import して型を取得
import { SomePlugin } from "some-library";
// window 経由ではなく、モジュールから直接 import する

// ✅ 正しい方法 4: 型定義がない場合は自分で定義する
declare module "untyped-library" {
  export interface UntypedClass {
    method(): void;
  }
}
```

## Fail Fast パターン

```typescript
/**
 * 引数の検証を関数の冒頭で行い、早期にエラーを発生させる
 */
function createUser(name: string, age: number): User {
  // 早期失敗: 引数を検証
  if (!name || name.trim().length === 0) {
    throw new ValidationError("name", "名前は必須です");
  }
  if (age < 0 || age > 150) {
    throw new ValidationError(
      "age",
      "年齢は 0 以上 150 以下である必要があります",
    );
  }

  return { name: name.trim(), age };
}
```

## JSON.parse の安全な処理

```typescript
/**
 * JSON.parse の結果を型安全に処理する
 * JSON.parse は unknown 型を返すため、型ガードまたはスキーマバリデーションが必須
 */
function parseConfig(json: string): AppConfig {
  const parsed: unknown = JSON.parse(json);

  // 型ガードで絞り込み
  if (
    typeof parsed === "object" &&
    parsed !== null &&
    "version" in parsed &&
    typeof (parsed as Record<string, unknown>).version === "number"
  ) {
    return parsed as AppConfig;
  }

  throw new Error("無効な設定ファイル形式");
}
```
