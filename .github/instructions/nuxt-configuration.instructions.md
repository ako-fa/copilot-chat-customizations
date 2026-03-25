---
description: 'Nuxt 4 の設定管理ガイド。環境別設定・プラグイン・ミドルウェア・モジュールの統一的な実装方法'
applyTo: 'nuxt.config.ts, app.config.ts, .env*, server/**, middleware/**, plugins/**'
---

# Nuxt 設定管理ガイドライン

Nuxt 4 の設定を安全かつ保守性高く管理するためのガイド。

## General Instructions

- 設定は `nuxt.config.ts` に集約し、環境別の分岐は `process.env.NODE_ENV` に限定する
- シークレット・API キーは環境変数に外出しし、コードに埋め込まない
- ランタイム設定は `app.config.ts` で管理し、ビルド時設定は `nuxt.config.ts` で分ける
- プラグイン・ミドルウェア・モジュールは自動検出を信頼し、ファイル命名規則を守る
- 複雑な設定ロジックは関数に抽出し、テスト可能に保つ

## nuxt.config.ts の主要セクション

| セクション      | 用途                                |
| --------------- | ----------------------------------- |
| `ssr`           | SSR 有効化（デフォルト true）       |
| `alias`         | パスエイリアス（`@/` は自動）       |
| `modules`       | Nuxt モジュール一覧                 |
| `routeRules`    | キャッシュ・プリレンダリング設定    |
| `nitro`         | サーバー・ビルド設定                |
| `runtimeConfig` | 実行時設定（`$config` でアクセス）  |
| `app`           | アプリレベル設定（head/build など） |

## app.config.ts

- アプリケーション固有の設定（テーマ・言語・UI 状態など）を管理する
- コンポーネント内では `useAppConfig()` でアクセスする

## 環境変数ルール

- `NUXT_PUBLIC_*` で始まる変数のみがクライアント側で利用可能
- シークレットは `NUXT_` で始まり、サーバーのみでアクセス可能
- `.env` ファイルに記載し、`.gitignore` に追加する

## Best Practices

- プラグイン実行順序が重要な場合は、ファイル名の数字 prefix でソートする（`00-global.ts` → `10-auth.ts`）
- ミドルウェアはページアクセス前にガード（認証など）を実装する
- `server/routes/` でカスタム API エンドポイントを作成し、外部 API 呼び出しを隔離する
- 本番環境では `routeRules` でコンテンツキャッシュを設定する
- 開発環境と本番環境で異なる設定が必要な場合は、`process.env.NODE_ENV` で分岐させる

## Verification

- 設定チェック: `pnpm run build` でビルド可能か確認する
- 環境変数: `NUXT_PUBLIC_*` がクライアントで利用できるか、`NUXT_` がサーバーのみかを確認する
- プラグイン: ブラウザコンソールで実行順序・エラーがないか確認する

## 参考リンク

- [Nuxt4 ドキュメント(設定ファイル)](https://nuxt.com/docs/4.x/getting-started/configuration)
