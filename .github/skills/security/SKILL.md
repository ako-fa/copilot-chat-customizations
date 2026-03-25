---
description: 'Nuxt 4 コーポレートサイトにおける Web セキュリティの実装コードパターン集'
---

# Security Skill

## HTTP セキュリティヘッダー（routeRules）

`nuxt.config.ts` の `routeRules` で全ページにセキュリティヘッダーを付与する。

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  routeRules: {
    '/**': {
      headers: {
        // HTTPS 強制（1 年間・サブドメイン含む・preload リスト登録）
        'Strict-Transport-Security':
          'max-age=31536000; includeSubDomains; preload',
        // クリックジャッキング対策: 同一オリジンのフレームのみ許可
        'X-Frame-Options': 'SAMEORIGIN',
        // MIME タイプスニッフィングを無効化
        'X-Content-Type-Options': 'nosniff',
        // レガシーブラウザ向け XSS フィルター
        'X-XSS-Protection': '1; mode=block',
        // Referer 情報を同一オリジン内のパスに限定
        'Referrer-Policy': 'strict-origin-when-cross-origin',
        // 不要な強力 API へのアクセスをブロック
        'Permissions-Policy':
          'camera=(), microphone=(), geolocation=(), payment=(), usb=(), interest-cohort=()',
      },
    },
  },
})
```

## Content Security Policy（CSP）

CSP ディレクティブ定義パターン。プロジェクトの実態に合わせて調整すること。

```ts
// nuxt.config.ts
// ---- CSP ディレクティブ ----
// 各ディレクティブのコメントは「なぜそのソースを許可するか」を必ず記載すること
const cspDirectives = [
  "default-src 'self'", // フォールバック: 同一オリジンのみ
  "script-src 'self'", // スクリプト: 同一オリジン（インライン禁止）
  "style-src 'self' 'unsafe-inline'", // スタイル: Tailwind のインライン一時許可（→ hash 移行目標）
  "img-src 'self' data:", // 画像: 同一オリジン + Base64
  "font-src 'self'", // フォント: 同一オリジン（@nuxt/fonts 使用時は追加）
  "connect-src 'self'", // Fetch/XHR: 同一オリジンのみ
  "frame-ancestors 'none'", // iframe 埋め込み元を全拒否（X-Frame-Options と二重防御）
  "form-action 'self'", // フォーム送信先: 同一オリジンのみ
  'upgrade-insecure-requests', // HTTP リクエストを HTTPS に自動アップグレード
].join('; ')

export default defineNuxtConfig({
  routeRules: {
    '/**': {
      headers: {
        'Content-Security-Policy': cspDirectives,
      },
    },
  },
})
```

> `'unsafe-eval'` は永続的に禁止。`'unsafe-inline'` は `script-src` から除去する目標を設定する。
> GSAP / Three.js でインラインスクリプトが必要な場合は `nonce` または `hash` 方式を採用する。

## runtimeConfig パターン（シークレット分離）

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  runtimeConfig: {
    // ---- サーバー専用（クライアントには絶対に露出しない） ----
    // NUXT_PUBLIC_ を付けると公開されるため絶対に付けないこと
    apiSecretKey: '', // .env: NUXT_API_SECRET_KEY
    // ---- 公開可能な設定（ビルド済み HTML に埋め込まれる） ----
    public: {
      siteUrl: '', // .env: NUXT_PUBLIC_SITE_URL
      gtmId: '', // .env: NUXT_PUBLIC_GTM_ID
    },
  },
})
```

## DOMPurify によるサニタイズパターン

`v-html` を使用せざるを得ない場合（CMS コンテンツ等）の唯一の許可パターン。

```bash
# 事前インストール
pnpm add dompurify
pnpm add -D @types/dompurify
```

```ts
// app/composables/useSafeHtml.ts
import DOMPurify from 'dompurify'

/**
 * 生 HTML 文字列を DOMPurify でサニタイズして返す Composable
 * @param rawHtml - サニタイズ前の生 HTML 文字列
 * @returns サニタイズ済み HTML 文字列（v-html に渡すためだけに使用する）
 */
export function useSafeHtml(rawHtml: Ref<string>) {
  // クライアントサイドのみ DOMPurify を実行する（SSR 非対応のため）
  const safeHtml = computed(() => {
    if (import.meta.server) return ''
    return DOMPurify.sanitize(rawHtml.value, {
      ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'ul', 'ol', 'li', 'a'],
      ALLOWED_ATTR: ['href', 'target', 'rel'],
    })
  })

  return { safeHtml }
}
```

```vue
<!-- 使用例: v-html には必ず safeHtml を渡す。rawHtml を直接渡すことを禁止する -->
<div v-html="safeHtml"></div>
```

## Subresource Integrity（SRI）パターン

外部 CDN からスクリプトを読み込む場合の必須パターン。

```bash
# SRI ハッシュの生成コマンド
openssl dgst -sha384 -binary lib.js | openssl base64 -A
```

```html
<!-- ✅ SRI ハッシュ付き（必須） -->
<script
  src="https://cdn.example.com/lib.min.js"
  integrity="sha384-ここに生成したハッシュを貼る"
  crossorigin="anonymous"
></script>
```

> 現在プロジェクトは GSAP・Three.js・Lenis をすべて npm パッケージとして管理しているため SRI は不要。
> CDN 方式に切り替える場合はこのパターンを必ず適用すること。

## .env テンプレート（.env.example）

```bash
# .env.example — 実際の値を含めず、キー名のみ列挙する（リポジトリに必ず含めること）

# ---- 公開可能な設定 ----
NUXT_PUBLIC_SITE_URL=
NUXT_PUBLIC_GTM_ID=

# ---- サーバー専用シークレット（NUXT_PUBLIC_ を絶対に付けないこと） ----
NUXT_API_SECRET_KEY=
```

```bash
# .gitignore に必ず含めること
.env
.env.local
.env.staging
.env.production
*.pem
*.key
```

## 脆弱性監査コマンド

```bash
# 既知の脆弱性を一覧表示する
pnpm audit

# High 以上の脆弱性があれば CI を失敗させる（CI/CD に組み込む）
pnpm audit --audit-level=high

# 自動修正可能な脆弱性を修正する（破壊的変更なし）
pnpm audit --fix

# 月次アップデート確認
pnpm outdated
```

## シークレット漏洩チェックコマンド

```bash
# gitleaks による Git 履歴のシークレット混入チェック
gitleaks detect --source . --verbose

# ビルド成果物のセキュリティヘッダー確認（本番 URL に対して実行）
curl -s -I https://your-domain.com \
  | grep -i "content-security-policy\|x-frame-options\|strict-transport\|x-content-type"
```

## .env の Git 履歴混入時の対処

```bash
# ⚠️ 必ず先にシークレットをローテーションしてから実行すること
# git revert だけでは履歴が残るため不十分

# 1. ファイルを Git 履歴から完全削除する
git filter-repo --path .env --invert-paths

# 2. リモートリポジトリへ強制プッシュする
git push origin --force --all

# 3. .gitignore への追記を確認する
grep -q "^\.env$" .gitignore || echo ".env" >> .gitignore
```

## ローカル HTTPS 設定（必要な場合のみ）

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  devServer: {
    https: {
      key: './certs/localhost-key.pem',
      cert: './certs/localhost.pem',
    },
  },
})
```

```bash
# mkcert を使ったローカル証明書の生成
mkcert -install
mkcert -key-file ./certs/localhost-key.pem -cert-file ./certs/localhost.pem localhost 127.0.0.1
```
