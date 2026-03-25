---
description: 'GSAP アニメーションの再利用可能なコードパターン集'
---

# GSAP Skill

GSAP 3.14.2 を使用したアニメーション実装の再利用可能なコードパターン。

> **前提**: 2025年以降、Webflow による GSAP 買収によりすべてのプラグインが完全無料。SplitText、MorphSVGPlugin、GSDevTools 等を含む全プラグインが npm パッケージ `gsap` に同梱。CDN 動的読み込みは不要。

## プラグイン登録パターン

```typescript
// plugins/gsap.ts
import gsap from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import { MorphSVGPlugin } from 'gsap/MorphSVGPlugin'
import { SplitText } from 'gsap/SplitText'

// プラグインを一括登録（一度だけ実行）
gsap.registerPlugin(ScrollTrigger, MorphSVGPlugin, SplitText)

// 開発環境では GSDevTools を動的に読み込み（バンドルサイズ削減）
const registerDevTools = async (): Promise<void> => {
  if (import.meta.dev && !import.meta.server) {
    const { GSDevTools } = await import('gsap/GSDevTools')
    gsap.registerPlugin(GSDevTools)
  }
}

export default defineNuxtPlugin(async (): Promise<void> => {
  await registerDevTools()
  // NOTE: provide は行わない。Composable では直接 import gsap from 'gsap' を使用する
})
```

## Import 規約

```typescript
// ✅ 正しい: default export で import
import gsap from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import { SplitText } from 'gsap/SplitText'

// ❌ 誤り: named export（gsap は default export）
// import { gsap } from 'gsap'

// ❌ 誤り: useNuxtApp() 経由は冗長で型安全性が低い
// const { $gsap } = useNuxtApp()

// ❌ 誤り: window 経由（型安全性が失われる）
// const SplitText = (window as any).SplitText
```

## ScrollTrigger Composable パターン

```typescript
// composables/useScrollFadeIn.ts
import gsap from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'

/**
 * スクロール連動のフェードインアニメーションを提供する Composable
 *
 * @param target - アニメーション対象要素の ref
 * @returns refresh - ScrollTrigger の再計算を実行する関数
 */
export const useScrollFadeIn = (
  target: Ref<HTMLElement | null>
): { refresh: () => void } => {
  let tl: gsap.core.Timeline | null = null

  const init = (): void => {
    if (!target.value) return

    tl = gsap.timeline({
      scrollTrigger: {
        trigger: target.value,
        start: 'top 75%',
        end: 'top 25%',
        scrub: 1,
        markers: import.meta.dev, // 開発環境のみマーカー表示
      },
    })

    tl.fromTo(
      target.value,
      { opacity: 0, y: 30 },
      { opacity: 1, y: 0, duration: 1 }
    )
  }

  const destroy = (): void => {
    tl?.kill()
    // 対象要素に紐づく ScrollTrigger のみ破棄する
    ScrollTrigger.getAll().forEach((trigger) => {
      if (trigger.vars.trigger === target.value) {
        trigger.kill()
      }
    })
  }

  onMounted(init)
  onUnmounted(destroy)

  return { refresh: () => ScrollTrigger.refresh() }
}
```

## Timeline パターン

```typescript
// 複数要素のシーケンシャルアニメーション
const animateElements = (elements: HTMLElement[]): gsap.core.Timeline => {
  const tl = gsap.timeline()

  // 第3引数でタイミングを制御（秒単位のオフセット）
  tl.to(elements[0], { opacity: 1, duration: 0.5 }, 0)
    .to(elements[1], { opacity: 1, duration: 0.5 }, 0.2)
    .to(elements[2], { opacity: 1, duration: 0.5 }, 0.4)

  return tl // 参照を返して外部から制御可能にする
}

// ❌ 避けるべき: 個別の Tween は管理が困難
// gsap.to(element1, { opacity: 1 })
// gsap.to(element2, { opacity: 1 })
```

## アニメーション状態管理パターン

```typescript
// composables/useAnimationState.ts
export const useAnimationState = (): {
  isAnimating: Readonly<Ref<boolean>>
  animationProgress: Readonly<Ref<number>>
  startAnimation: (duration: number) => Promise<void>
} => {
  const isAnimating = ref(false)
  const animationProgress = ref(0)

  const startAnimation = async (duration: number): Promise<void> => {
    if (isAnimating.value) return

    isAnimating.value = true
    animationProgress.value = 0

    return new Promise((resolve) => {
      gsap.to(
        {},
        {
          duration,
          onUpdate() {
            animationProgress.value = this.progress()
          },
          onComplete() {
            isAnimating.value = false
            resolve()
          },
        }
      )
    })
  }

  return {
    isAnimating: readonly(isAnimating),
    animationProgress: readonly(animationProgress),
    startAnimation,
  }
}
```

## GPU 加速ルール

```typescript
// ✅ GPU 加速されるプロパティのみアニメーション
gsap.to(element, {
  x: 100, // transform: translateX → GPU 加速
  y: 50, // transform: translateY → GPU 加速
  opacity: 0.5, // opacity → GPU 加速
  scale: 1.2, // transform: scale → GPU 加速
  rotation: 45, // transform: rotate → GPU 加速
  duration: 1,
})

// ❌ リフロー発生プロパティは使用禁止
// gsap.to(element, { left: '100px', top: '100px', width: '200px' })
```

## markers オプション規約

```typescript
// 開発環境では自動的にマーカーを表示
ScrollTrigger.create({
  trigger: element,
  markers: import.meta.dev, // 開発: true、本番: false
})
```
