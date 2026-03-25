---
description: 'Nuxt 4 コーポレートサイトにおける Web セキュリティの実装・審査・保守ガイドライン'
applyTo: 'nuxt.config.ts, app.config.ts, server/**, app/**/*.vue, app/**/*.ts, app/plugins/**/*.ts, public/**, .env*'
---

# セキュリティガイドライン

> **実装コードパターンは `.github/skills/security/SKILL.md` を参照すること。**
> 本ファイルはルール・禁止事項・判断基準のみを定義する。

## 前提・スコープ

本ガイドラインは **Nuxt 4 + 静的サイト生成（`nuxt generate`）** を前提とする。
SSR 固有の対策は「SSR のみ」と明示した上で区別する。

---

## 1. HTTP セキュリティヘッダー

**ルール**: 必須ヘッダーをすべて `nuxt.config.ts` の `routeRules['/**'].headers` に定義すること。
実装コードは `SKILL.md` の「HTTP セキュリティヘッダー（routeRules）」を使用する。

必須ヘッダー一覧（すべて ✅ が揃うまでリリースを禁止する）:

| ヘッダー                    | 確認コマンド                                           |
| --------------------------- | ------------------------------------------------------ |
| `Strict-Transport-Security` | `curl -I https://domain.com`                           |
| `X-Frame-Options`           | Chrome DevTools > Network > Response Headers           |
| `X-Content-Type-Options`    | 同上                                                   |
| `Referrer-Policy`           | 同上                                                   |
| `Permissions-Policy`        | 同上                                                   |
| `Content-Security-Policy`   | [CSP Evaluator](https://csp-evaluator.withgoogle.com/) |

### CSP の追加ルール

- `script-src` への `'unsafe-inline'` は禁止。`nonce` または `hash` 方式を採用すること。
- `'unsafe-eval'` は永続的に禁止。
- `style-src 'unsafe-inline'` は一時許可のみ（Tailwind 対応中）。ハッシュ移行を目標とする。
- 外部ドメインのリソースを読み込む場合は、対応する CSP ディレクティブにドメインを追加すること。
- CSP 変更時は必ず `Content-Security-Policy-Report-Only` で動作確認してから本適用する。

---

## 2. 環境変数・シークレット管理

**ルール**:

- `NUXT_PUBLIC_` プレフィックスは公開可能な値専用とする。API キー・トークンに付けることを**絶対に禁止する**。
- `runtimeConfig.public` への値は `nuxt generate` 後に HTML に埋め込まれる。シークレット混入を**絶対に禁止する**。
- `.env.example` をキー名のみで作成し、リポジトリに必ず含めること。
- `.env` / `.env.local` / `.env.staging` / `.env.production` / `*.pem` / `*.key` を `.gitignore` に必ず含めること。

**シークレット誤コミット時**: `git revert` のみでは不十分。`git filter-repo` による履歴完全削除が必須。
手順は `SKILL.md` の「.env の Git 履歴混入時の対処」を参照すること。

---

## 3. 依存パッケージの脆弱性管理

**ルール**:

- CI/CD パイプラインに `pnpm audit --audit-level=high` を組み込む（High 以上でビルドをブロックする）。
- 月次で `pnpm outdated` を確認し、マイナー・パッチアップデートを適用する。
- 依存関係の更新は専用ブランチで行い、ビルドとテストが通ることを確認してからマージする。

コマンドは `SKILL.md` の「脆弱性監査コマンド」を参照すること。

---

## 4. クッキーセキュリティ

サードパーティスクリプト（GA・GTM 等）が発行するクッキーも含め、`Set-Cookie` ヘッダーには
`HttpOnly; Secure; SameSite=Strict` を必ず付与すること（ホスティング側の設定）。

---

## 5. 外部リソース（サードパーティスクリプト）

**ルール**:

- 外部 CDN からスクリプトを読み込む場合は SRI（Subresource Integrity）ハッシュを必ず付与すること。
- GSAP・Three.js・Lenis は現在 npm パッケージとして管理しているため SRI は不要だが、CDN 方式に切り替える場合は適用必須とする。

実装パターンは `SKILL.md` の「Subresource Integrity（SRI）パターン」を参照すること。

---

## 6. robots.txt / sitemap.xml

**ルール**:

- `public/_robots.txt` に `/admin/`・`/staging/`・`/.env`・`/api/` を `Disallow` で列挙すること。
- ステージング環境では `Disallow: /` を指定し、検索エンジンにインデックスさせないこと。
- `sitemap.xml` に存在しないパスや内部パスを含めないこと。

---

## 7. XSS 対策

**ルール**:

- `v-html` は**原則禁止**。使用する場合は信頼できるソースであることを文書化し、レビューで承認を得ること。
- `v-html` を使用する場合は必ず DOMPurify でサニタイズすること。

実装パターンは `SKILL.md` の「DOMPurify によるサニタイズパターン」を参照すること。

---

## 8. ファイルアップロード（将来実装時の制約）

- アップロード可能な MIME タイプはホワイトリストで制限すること。
- ファイルサイズの上限を明示的に設定すること（デフォルト値への依存を禁止する）。
- アップロードファイルを `public/` に直接配置することを禁止する。

---

## 9. nuxt-security モジュールの導入判断基準

| 条件                                   | 判断                                                    |
| -------------------------------------- | ------------------------------------------------------- |
| Nuxt 4 正式対応済みかつ SSR モード使用 | 積極的に導入を検討する                                  |
| Nuxt 4 正式対応済みかつ静的生成のみ    | `routeRules` で代替可能なため任意                       |
| Nuxt 4 未対応                          | **導入禁止**。`routeRules` + ホスティング設定で対応する |

互換性は必ず [公式ドキュメント](https://nuxt-security.vercel.app/) で確認してから判断すること。

---

## 10. 禁止事項まとめ

| 禁止事項                                                    | 理由                                                 |
| ----------------------------------------------------------- | ---------------------------------------------------- |
| `NUXT_PUBLIC_` プレフィックスを API キーに付ける            | クライアントバンドルに混入しセキュリティリスクとなる |
| `v-html` に未サニタイズ文字列を渡す                         | XSS の直接的な原因となる                             |
| `.env` をリポジトリにコミットする                           | シークレットの漏洩につながる                         |
| `Content-Security-Policy` に `'unsafe-eval'` を永続的に残す | eval 系の攻撃を許容することになる                    |
| HTTP のみで本番サイトを公開する                             | 通信の盗聴・改ざんのリスクがある                     |
| 外部スクリプトを SRI なしで CDN から読み込む                | サプライチェーン攻撃の踏み台となる                   |
