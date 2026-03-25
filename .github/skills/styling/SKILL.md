---
description: 'Tailwind CSS + SCSS スタイリングの再利用可能なコードパターン集'
---

# Styling Skill

Tailwind CSS v4 と SCSS の役割分担に基づくスタイリングの再利用可能なコードパターン。

## パターン 1: Tailwind レイアウト + SCSS ディテール

```vue
<template>
  <div class="grid grid-cols-1 gap-6 md:grid-cols-2">
    <Card class="card-item" />
  </div>
</template>

<style scoped lang="scss">
// Tailwind でレイアウトを定義 → SCSS でディテールを追加
.card-item {
  &:hover {
    @keyframes cardFlip {
      0% {
        transform: rotateY(0);
        opacity: 1;
      }
      50% {
        opacity: 0.8;
      }
      100% {
        transform: rotateY(180deg);
        opacity: 1;
      }
    }
    animation: cardFlip 0.8s cubic-bezier(0.34, 1.56, 0.64, 1) forwards;
  }
}
</style>
```

## パターン 2: 条件付きクラス分岐

```vue
<template>
  <div
    :class="[
      'flex items-center gap-4 rounded-lg p-4', // Tailwind（常に適用）
      isActive && 'border-2 border-blue-500 bg-blue-100', // Tailwind（条件）
      { 'state-active': isActive }, // SCSS クラス（複雑な状態）
    ]"
  >
    <span class="font-semibold">Status</span>
  </div>
</template>

<style scoped lang="scss">
.state-active {
  &::before {
    content: '';
    position: absolute;
    animation: pulseRing 2s infinite;
  }

  @keyframes pulseRing {
    0% {
      transform: scale(1);
      opacity: 1;
    }
    100% {
      transform: scale(1.2);
      opacity: 0;
    }
  }
}
</style>
```

## パターン 3: GSAP との連携

```vue
<script setup lang="ts">
const elementRef = ref<HTMLElement>()
// GSAP で値を制御する場合
</script>

<template>
  <div
    ref="elementRef"
    class="rounded-lg bg-white p-6 shadow-lg md:p-8"
    :style="{ opacity: 0, transform: 'translateY(50px)' }"
  >
    <h3 class="mb-4 text-2xl font-bold text-gray-900">タイトル</h3>
    <p class="leading-relaxed text-gray-600">説明文</p>
  </div>
</template>

<style scoped lang="scss">
// 初期値は :style で設定、GSAP が値を制御する
// will-change などのパフォーマンス最適化のみ SCSS で記述可能
</style>
```

## パターン 4: GSAP 状態クラス

```scss
.element {
  transition: none; // GSAP 制御時は CSS transition を無効化

  &.is-animating {
    will-change: transform, opacity;
    pointer-events: none;
  }

  &.is-complete {
    cursor: pointer;
    pointer-events: auto;
  }
}
```

## パターン 5: SCSS 変数と計算

```scss
$primary-color: #2563eb;
$spacing-unit: 1rem;

.hero-section {
  padding: $spacing-unit * 3;
  background: linear-gradient(
    135deg,
    $primary-color,
    lighten($primary-color, 20%)
  );
  font-size: clamp(1.5rem, 5vw, 3rem);
  max-width: calc(100% - 2rem);
}
```

## パターン 6: 複雑なセレクタ・ネスト

```scss
.card {
  border-radius: 0.5rem;

  &:hover {
    .card__title {
      color: $primary-color;
    }
    .card__image {
      transform: scale(1.05);
    }
  }

  &.is-featured {
    &:hover {
      .card__badge {
        opacity: 1;
      }
    }
  }

  // ネストは 3 階層まで推奨
  > .card__header {
    display: flex;
    gap: 1rem;
  }
}
```

## Tailwind 担当領域（一覧）

| 用途               | Tailwind クラス例                                      |
| ------------------ | ------------------------------------------------------ |
| グリッドレイアウト | `grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3`       |
| Flex レイアウト    | `flex flex-col items-center justify-between`           |
| スペーシング       | `p-6 md:p-8`, `gap-4`, `mx-auto`                       |
| タイポグラフィ     | `text-2xl font-bold leading-relaxed text-gray-900`     |
| 背景・ボーダー     | `bg-white border border-gray-300 rounded-lg shadow-lg` |
| ホバー状態         | `hover:bg-blue-600 transition-colors duration-200`     |
| レスポンシブ       | `md:grid-cols-2 lg:text-5xl`                           |

## SCSS 担当領域（一覧）

| 用途                 | 使用場面                                 |
| -------------------- | ---------------------------------------- |
| `@keyframes`         | 複数ステップのアニメーション             |
| 複雑なセレクタ       | 親要素の状態に基づく子要素のスタイル変更 |
| `calc()` / `clamp()` | 動的に計算すべき値                       |
| SCSS 変数            | 複数箇所で使用する計算値                 |
| scoped スタイル      | Vue コンポーネント固有の複雑なスタイル   |

## ファイル構成

```
app/
├── assets/
│   └── style/
│       ├── scss/
│       │   └── variables.scss       # プロジェクト全体の SCSS 変数
│       └── css/
│           └── tailwind.css         # Tailwind エントリーポイント
├── components/
│   └── ui/
│       └── BaseButton/
│           └── BaseButton.vue       # Tailwind + scoped SCSS
```

> **注意**: ファイル構成のパスは `app/`（`src/` ではない）。
