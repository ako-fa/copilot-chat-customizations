---
description: "セキュリティ実装パターンと監査チェックリスト"
---

# Security Skill

任意の Web アプリケーションで利用できる、実装パターンと監査手順を定義する。

## OWASP Top 10 監査チェックリスト

- A01 Broken Access Control: 認可判定をサーバー側で強制し、ID 直接参照を禁止する
- A02 Cryptographic Failures: 平文保存を禁止し、機密データを暗号化する
- A03 Injection: SQL/NoSQL/OS コマンドをパラメータ化し、文字列連結を禁止する
- A04 Insecure Design: 脅威モデリングとセキュアデフォルトを設計段階で実施する
- A05 Security Misconfiguration: 不要機能を無効化し、設定差分を環境ごとに管理する
- A06 Vulnerable and Outdated Components: 依存関係の脆弱性を継続監査する
- A07 Identification and Authentication Failures: 認証強度、セッション管理、再認証要件を定義する
- A08 Software and Data Integrity Failures: 依存物と配布物の整合性を検証する
- A09 Security Logging and Monitoring Failures: 監査ログを保持し、検知と通知を自動化する
- A10 Server-Side Request Forgery: 外部 URL 取得は許可リスト方式で制限する

## セキュリティヘッダー設定

以下を HTTP レスポンスヘッダーとして設定する。

- `Strict-Transport-Security: max-age=31536000; includeSubDomains; preload`
- `Content-Security-Policy: default-src 'self'; frame-ancestors 'none'; form-action 'self'`
- `X-Frame-Options: DENY` または `SAMEORIGIN`
- `X-Content-Type-Options: nosniff`
- `Referrer-Policy: strict-origin-when-cross-origin`
- `Permissions-Policy: camera=(), microphone=(), geolocation=()`

## CSP の方針

- `script-src` で `unsafe-eval` を禁止する
- 可能な限り `unsafe-inline` を使わず、`nonce` または `hash` を使用する
- 外部ドメイン許可は最小化し、理由を設定ファイル上に明記する
- レポートモード（`Content-Security-Policy-Report-Only`）で段階導入し、本番で強制モードへ移行する

## XSS 対策（汎用）

- ユーザー入力を HTML として直接描画しない
- HTML 表示が必要な場合は、信頼済みサニタイザで許可タグのみ通過させる
- テンプレートエンジンの自動エスケープを無効化しない
- URL を属性に埋め込む場合はスキーム検証（`http`/`https` など）を行う

## 入力バリデーションパターン

- 境界（HTTP、CLI、メッセージキュー）で必ず検証する
- 許可リスト方式を採用し、フォーマット・長さ・範囲を明示する
- 検証失敗時は fail fast で処理を打ち切る
- 正規化（トリム、大小文字統一）後の値を業務処理へ渡す

### 例

```typescript
type CreateUserInput = {
  name: string;
  email: string;
};

export function validateCreateUserInput(
  input: CreateUserInput,
): CreateUserInput {
  const name = input.name.trim();
  const email = input.email.trim().toLowerCase();

  if (name.length === 0 || name.length > 100) {
    throw new Error("invalid name");
  }
  if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
    throw new Error("invalid email");
  }

  return { name, email };
}
```

## 認証・認可の実装パターン

- 認証は信頼できる ID プロバイダ連携または十分な強度のパスワード認証を採用する
- パスワードはメモリハードなアルゴリズム（Argon2id / bcrypt）でハッシュ化する
- セッション ID / トークンは HTTP Only・Secure・SameSite 属性を付与する
- 認可はエンドポイント単位ではなくリソース単位で判定する
- 権限不足時は詳細情報を返さず、最小限のエラーレスポンスにする

## 暗号化の方針

- 通信は TLS 1.2 以上を必須化する
- 保存データ暗号化は標準ライブラリまたは実績ある実装を使う
- 独自暗号アルゴリズムを実装しない
- 鍵管理はアプリ外部（KMS/HSM/Secret Manager）に分離する
- 鍵・トークン・秘密値をログへ出力しない

## 依存関係の脆弱性管理

- 依存関係を定期スキャンし、High/Critical を優先修正する
- SBOM を生成し、サプライチェーン可視性を維持する
- 自動更新はテスト付きで段階適用し、破壊的更新を監視する
- 使っていない依存関係を削除する

## シークレット管理

- シークレットをソースコードへ直書きしない
- 環境変数・秘密情報管理サービスで注入する
- `.env.example` にはキー名のみ記載し、値を入れない
- 漏洩時は即時ローテーションし、失効範囲を監査する

## 継続監査チェック

- CI で SAST、依存脆弱性スキャン、シークレットスキャンを実行しているか
- 本番のレスポンスヘッダーを定期検査しているか
- 監査ログが改ざん耐性を持つ保管先へ送信されているか
- インシデント対応手順（封じ込め、根絶、復旧、事後分析）が文書化されているか
