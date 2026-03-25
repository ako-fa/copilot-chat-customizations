---
description: 'Nuxt/Vue フロントエンドのパフォーマンス最適化ガイド（描画・ネットワーク・JS 実行コスト削減）'
applyTo: 'app/**/*.vue, app/**/*.ts, app/plugins/**/*.ts, app/composables/**/*.ts'
---

# パフォーマンス最適化ガイドライン

LCP/CLS/FID を意識し、描画・ネットワーク・スクリプトのコストを最小化するための指針。

## General Instructions

- 初期表示を最優先し、折りたたみ外のリソースは遅延読込する
- 依存を減らし、重いライブラリは代替・部分 import を検討する
- レイアウトシフトを防ぐため、画像・動画には寸法を指定しプレースホルダーを用意する
- 再描画が高頻度な箇所はリアクティブ依存を最小限にし、計算を `computed` / メモ化に寄せる
- 計測なしの最適化を禁止。Before/After を計測してから採用する

## Best Practices

### ネットワーク

- 画像は適切なサイズとフォーマット（WebP/AVIF）で提供し、`loading="lazy"` を設定する
- ルート分割を活用し、ページ単位で遅延ロードする（`defineAsyncComponent` や Nuxt の自動コード分割）
- API リクエストはキャッシュ可能なものを `stale-while-revalidate` などで再利用する

### 描画とレイアウト

- アニメーションは `transform` と `opacity` に限定し、`left/top` でのレイアウト再計算を避ける
- スクロール/リサイズなど高頻度イベントは `requestAnimationFrame`、`throttle`/`debounce` で制御する
- 大量リストはバーチャルスクロールを検討し、DOM ノード数を抑える

### JavaScript 実行コスト

- 不要な `watch` を避け、派生値は `computed` に集約する
- 大きなオブジェクトのリアクティブ化を避け、必要なフィールドだけ `ref` 化する
- `console` やデバッグ用コードは本番ビルドで削除する

## Common Patterns

### 遅延ローディング（良い例）

```typescript
import { defineAsyncComponent } from 'vue'

const AsyncHero = defineAsyncComponent(
  () => import('@/components/features/HeroSection.vue')
)
```

### 高頻度イベントの抑制（良い例）

```typescript
import { useThrottleFn } from '@vueuse/core'

const onScroll = useThrottleFn(() => {
  // スクロールに応じた処理
}, 100)

onMounted(() => window.addEventListener('scroll', onScroll))
onUnmounted(() => window.removeEventListener('scroll', onScroll))
```

### 悪い例：レイアウトスラッシング

```typescript
window.addEventListener('scroll', () => {
  // 毎フレームで DOM 測定と書き込みを交互に行う ❌
  const h = document.body.clientHeight
  document.body.style.height = `${h + 1}px`
})
```

## Verification

- ビルド最適化: `pnpm run build`
- バンドル確認: `pnpm run build` 出力サイズを確認し、不要バンドルがないかチェック
- パフォーマンス計測: Chrome DevTools Performance/Network、Lighthouse で LCP/CLS/FID を計測（変更前後を比較）
