---
description: "GSAP、Three.js、Lenis を使用したアニメーションと3Dグラフィックスの宣言的ルール"
applyTo: "app/components/**/*.vue, app/composables/**/*.ts, app/plugins/**/*.ts, app/pages/**/*.vue"
---

# アニメーション・グラフィックス ガイドライン

## 基本原則

- すべてのアニメーションは Composable として実装する（コンポーネントに直接書かない）
- `onMounted` でアニメーションを初期化し、`onUnmounted` で確実に破棄する
- `readonly()` で外部公開値を保護する
- パフォーマンス測定を定期的に行い、60fps を維持する
- 対象技術は GSAP（3.14.2）、Three.js（0.182.0）、Lenis（1.3.16）とし、用途はそれぞれアニメーション全般・3D グラフィックス・スムーズスクロールに限定する
- GSAP は 2025 年に Webflow に買収され、すべてのプラグインが完全無料化されているため、CDN 動的読み込みを行わない

## 必須ルール

### GSAP

- `import gsap from 'gsap'`（default export）を使用する。`useNuxtApp()` 経由は非推奨
- 複数のアニメーションは Timeline でまとめ、参照を保持する
- `onUnmounted` で `timeline.kill()` と `ScrollTrigger.kill()` を実行する
- `markers: import.meta.dev` で開発環境のみマーカーを表示する
- GPU 加速されるプロパティのみをアニメーションする（`x`, `y`, `opacity`, `scale`, `rotation`）
- `left`, `top`, `width`, `height` のアニメーションは禁止（リフロー発生）

### Three.js

- `onUnmounted` で `geometry.dispose()`, `material.dispose()`, `renderer.dispose()` を実行する
- ジオメトリは可能な限り再利用する
- レンダリングループは `requestAnimationFrame` で管理し、`onUnmounted` で `cancelAnimationFrame` する
- リサイズハンドラーの登録と解除を行う

### Lenis

- `app/plugins/lenis.ts` が唯一の初期化ポイント。`'lenis/nuxt'` モジュールを `nuxt.config.ts` に再登録することは**絶対に禁止**する（二重初期化が発生する）
- `import Lenis from 'lenis'` を使用する。旧パッケージ名 `@studio-freight/lenis` は廃止済み
- `autoRaf: true` と GSAP ticker を併用しない（二重 RAF になる）
- GSAP ScrollTrigger と同期する場合は `lenis.on('scroll', ScrollTrigger.update)` + `gsap.ticker.add()` パターンを使用する

### Lenis 廃止オプション

| 旧オプション       | 代替                        |
| ------------------ | --------------------------- |
| `direction`        | `orientation` を使用        |
| `smooth`           | `smoothWheel` を使用        |
| `smoothTouch`      | `syncTouch` を使用          |
| `gestureDirection` | `gestureOrientation` を使用 |

## ファイル構成

```
app/
 composables/
   ├── use*Animation.ts     # スクロール・パラレックス等
   ├── useThreeScene.ts     # Three.js シーン管理
   └── useLenisScroll.ts    # Lenis 操作
 plugins/
   ├── gsap.ts              # GSAP プラグイン登録
   └── lenis.ts             # Lenis 初期化 + GSAP 同期
```

## コードパターン参照

- GSAP パターン: `#skill:gsap`
- Three.js パターン: `#skill:threejs`
- Lenis パターン: `#skill:lenis`

## ワークフロー参照

- アニメーション実装ワークフロー: `#agent:animation`

## 参考リンク

- [GSAP 公式ドキュメント](https://gsap.com/docs/v3/)
- [GSAP ScrollTrigger](https://gsap.com/docs/v3/Plugins/ScrollTrigger/)
- [Three.js 公式ドキュメント](https://threejs.org/docs/)
- [Lenis GitHub](https://github.com/darkroomengineering/lenis)

## チェックリスト

- GSAP アニメーションを `useGsap` などの composable で宣言的に記述しているか
- Three.js のシーン管理を `useThree` / `useThreeScene` composable に集約しているか
- Lenis の設定と初期化を `app/plugins/lenis.ts` と `useLenis` / `useLenisScroll` composable で一元管理しているか
- `onUnmounted` で GSAP Timeline / ScrollTrigger、Three.js の `dispose()`、`cancelAnimationFrame`、イベントリスナー解除を実装しているか
- GSAP で `x`, `y`, `opacity`, `scale`, `rotation` を中心にアニメーションし、`left`, `top`, `width`, `height` を使用していないか
- Lenis で二重初期化と二重 RAF（`autoRaf: true` と GSAP ticker 併用）を避けているか
- 開発環境のみ `markers: import.meta.dev` を有効化し、本番環境に不要なデバッグ設定を残していないか
- パフォーマンス計測を実施し、60fps の維持を確認しているか
