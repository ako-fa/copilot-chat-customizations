---
description: "Lenis スムーズスクロールの再利用可能なコードパターン集"
---

# Lenis Skill

Lenis 1.3.16 を使用したスムーズスクロール実装の再利用可能なコードパターン。

> **重要**: 本プロジェクトでは `nuxt.config.ts` に `'lenis/nuxt'` モジュールを**登録していない**。過去に `lenis/nuxt` モジュールと `plugins/lenis.ts` の二重初期化バグが発生したため、モジュールを削除し `app/plugins/lenis.ts` による手動プラグイン初期化に一本化している。`'lenis/nuxt'` をモジュールとして再登録することは**絶対に禁止**する。詳細は `doc/investigation/defect_2026-03-04.md` を参照すること。

## パッケージ名の注意

```typescript
// ✅ 現在のパッケージ名
import Lenis from "lenis";

// ❌ 廃止されたパッケージ名（使用禁止）
// import Lenis from '@studio-freight/lenis'
```

## 推奨 CSS の import

```typescript
// Lenis が提供する推奨 CSS
// html.lenis クラスの height: auto などが設定される
import "lenis/dist/lenis.css";
```

## GSAP ScrollTrigger との同期パターン（公式推奨）

```typescript
// plugins/lenis.ts
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
import Lenis from "lenis";
import "lenis/dist/lenis.css";

export default defineNuxtPlugin(() => {
  // SSR では実行しない
  if (import.meta.server) return;

  // ブラウザのスクロール位置自動復元を無効化する（Lenis が手動管理するため）
  if ("scrollRestoration" in history) {
    history.scrollRestoration = "manual";
  }

  const lenis = new Lenis({
    duration: 1.2,
    easing: (t): number => Math.min(1, 1.001 - Math.pow(2, -10 * t)),
    orientation: "vertical",
    smoothWheel: true,
    syncTouch: true,
    syncTouchLerp: 0.1,
  });

  // 同期ポイント 1: Lenis のスクロールイベントを ScrollTrigger に通知
  lenis.on("scroll", ScrollTrigger.update);

  // 同期ポイント 2: GSAP ticker で Lenis の RAF を駆動
  // （手動 requestAnimationFrame は不要）
  gsap.ticker.add((time: number): void => {
    lenis.raf(time * 1000); // GSAP ticker の time は秒単位 → ミリ秒に変換
  });

  // 同期ポイント 3: ラグ平滑化を無効化（Lenis 側で補間するため）
  gsap.ticker.lagSmoothing(0);

  // ページ遷移時にスクロール位置をリセット
  const router = useRouter();
  router.afterEach((): void => {
    lenis.scrollTo(0, { immediate: true });
  });

  return {
    provide: {
      lenis, // Composable では useNuxtApp() 経由でアクセス
    },
  };
});
```

> **注意**: `autoRaf: true` オプションと GSAP ticker を**併用しないこと**。両方有効にすると Lenis が二重に RAF を実行する。

## Lenis Composable パターン

Lenis はプロジェクト全体で単一インスタンスとして管理するため、Composable では `useNuxtApp()` 経由でアクセスする（GSAP とは異なり、共有インスタンスへの参照が必要なため）。

```typescript
// composables/useLenisScroll.ts

/**
 * Lenis スムーズスクロール操作を提供する Composable
 *
 * @returns smoothScrollTo, scrollToImmediate, stop, start の各関数
 */
export const useLenisScroll = (): {
  smoothScrollTo: (
    target: string | HTMLElement | number,
    offset?: number,
  ) => void;
  scrollToImmediate: (offset: number) => void;
  stop: () => void;
  start: () => void;
} => {
  const { $lenis } = useNuxtApp();

  // SSR 時またはプラグイン未初期化時は何もしない
  if (import.meta.server || !$lenis) {
    return {
      smoothScrollTo: (): void => {},
      scrollToImmediate: (): void => {},
      stop: (): void => {},
      start: (): void => {},
    };
  }

  const smoothScrollTo = (
    target: string | HTMLElement | number,
    offset = 0,
  ): void => {
    $lenis.scrollTo(target, { offset, immediate: false });
  };

  const scrollToImmediate = (offset: number): void => {
    $lenis.scrollTo(offset, { immediate: true });
  };

  const stop = (): void => {
    $lenis.stop();
  };
  const start = (): void => {
    $lenis.start();
  };

  return { smoothScrollTo, scrollToImmediate, stop, start };
};
```

## 廃止・変更されたオプション

| 旧オプション       | 状態     | 代替                        |
| ------------------ | -------- | --------------------------- |
| `direction`        | 名称変更 | `orientation` を使用        |
| `smooth`           | 廃止     | `smoothWheel` を使用        |
| `smoothTouch`      | 廃止     | `syncTouch` を使用          |
| `gestureDirection` | 名称変更 | `gestureOrientation` を使用 |
| `touchMultiplier`  | 変更     | デフォルト値が 2 → 1 に変更 |
