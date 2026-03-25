---
description: "フロントエンドアセット（画像・フォント・ビデオ）の管理・最適化・配信ガイド"
applyTo: "public/**, app/assets/**, **/*.{jpg,jpeg,png,gif,webp,svg,woff2,ttf,mp4}"
---

# アセット管理ガイドライン

# 1. gsap.set の mock.calls → as GsapCallArgs[]------持するための指針。

## General Instructions

- アセットは `public/` または `app/assets/` に配置し、用途別にディレクトリで整理する
- 画像は複数サイズとフォーマット（WebP/AVIF）を提供し、`<picture>` で段階的フォールバック
- フォントは WOFF2 のみで配信し、`font-display: swap` でローディング間の表示を制御する
- ビデオは適切な codec と解像度で複数提供し、外部 CDN（YouTube など）の利用を検討する
- すべてのアセットに `alt` テキストまたは `title` をメタデータとして記録する

## ディレクトリ構成

```
public/
 images/
   ├── hero/              # ヒーロー画像
   ├── logo/              # ロゴ・アイコン
   ├── social/            # OGP 画像など
   └── favicon/           # Favicon 各サイズ
 fonts/                 # 日本語フォント（WOFF2）
   ├── noto-sans-jp/
   └── noto-serif-jp/
 videos/                # 背景ビデオなど（最小限に）

app/assets/
 images/                # Vue コンポーネントで使用する画像
 style/                 # グローバル SCSS/CSS
 icons/                 # SVG アイコンコンポーネント化
```

## 画像ルール

- 元画像は高解像度（2x 以上）で用意し、複数サイズを生成する
- JPEG/PNG は ImageOptim または TinyPNG で最適化する
- WebP/AVIF への変換は `cwebp`/`avifenc` で実施する
- SVG はテキストエディタで手作業で最適化する（ID やメタデータ除去）
- `<img>` / `<NuxtImg>` に `width`/`height` を必ず指定する（CLS 防止）
- above-the-fold（ファーストビュー）には `loading="eager"` を設定する
- below-the-fold（スクロール先）には `loading="lazy"` を設定する（NuxtImg のデフォルト）

## フォントルール

- WOFF2 フォーマットのみ使用する
- `font-display: swap` を必ず設定する
- ウェイトは 400（regular）と 700（bold）のみを基本とする

## ビデオルール

- 複数コーデック（mp4 + webm）を提供する
- `preload="none"` でデフォルトの事前読み込みを無効にする
- ポスター画像を必ず設定する
- 外部ホスト（YouTube など）の利用を優先的に検討する

## Verification

- 画像最適化: `pnpm run build` 後、`.nuxt/dist` のアセットサイズを確認する
- 寸法指定: `<img>` / `<NuxtImg>` に `width`/`height` が指定されているか確認する
- alt テキスト: すべての装飾的でない画像に `alt` 属性があるか確認する
- 本番配信: CDN キャッシュ設定（Cache-Control ヘッダー）が正しく設定されているか確認する
