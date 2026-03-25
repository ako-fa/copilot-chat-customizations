---
description: 'リファクタリングの再利用可能なコードパターン集（functional コンポーネント→Composable 変換、依存正規化など）'
---

# Refactor Skill

既存コードの構造改善に使用するリファクタリングパターン집。

## パターン 1: functional コンポーネント → Composable 変換

見た目を伴わない機能コンポーネントを Composable に変換する。

### 変換前（アンチパターン）

```vue
<!-- components/functional/KeyBind.vue -->
<template>
  <slot />
</template>

<script setup lang="ts">
/**
 * キーバインドを管理するコンポーネント（アンチパターン）
 * slot を通すだけで <template> が無駄であるため Composable に変換すべき
 */
const handleKeyDown = (e: KeyboardEvent) => {
  /* ... */
}
onMounted(() => window.addEventListener('keydown', handleKeyDown))
onUnmounted(() => window.removeEventListener('keydown', handleKeyDown))
</script>
```

### 変換後（推奨パターン）

```typescript
// composables/useKeyBind.ts

/**
 * キーバインドを管理する Composable
 * 指定したキーが押された際にコールバックを実行する
 * @param keys - 監視するキー名の配列（toLowerCase で比較）
 * @param callback - キーが押された際に実行する関数
 */
export const useKeyBind = (keys: string[], callback?: () => void) => {
  const pressedKeys = ref<Set<string>>(new Set())

  const handleKeyDown = (event: KeyboardEvent): void => {
    const key = event.key.toLowerCase()
    if (keys.some((k) => k === key)) {
      // デフォルト動作（スクロール等）を抑制し、コールバックを実行する
      event.preventDefault()
      pressedKeys.value.add(key)
      callback?.()
    }
  }

  const handleKeyUp = (event: KeyboardEvent): void => {
    pressedKeys.value.delete(event.key.toLowerCase())
  }

  onMounted(() => {
    window.addEventListener('keydown', handleKeyDown)
    window.addEventListener('keyup', handleKeyUp)
  })

  onUnmounted(() => {
    // イベントリスナーを必ず解除してメモリリークを防ぐ
    window.removeEventListener('keydown', handleKeyDown)
    window.removeEventListener('keyup', handleKeyUp)
  })

  return { pressedKeys: readonly(pressedKeys) }
}
```

**変換理由**:

- `<template><slot /></template>` のみの Vue コンポーネントは無駄な VDOM ノードを生成する
- `use*` プリフィックスで自動インポートされ、API が呼び出し側で簡潔になる
- ライフサイクル管理が明示的になり、メモリリークを防ぎやすくなる

---

## パターン 2: 重複ロジックの Composable 抽出

複数コンポーネントで同一のロジックが重複している場合に抽出する。

### 抽出前（重複あり）

```vue
<!-- ComponentA.vue -->
<script setup lang="ts">
const isVisible = ref(false)
const toggle = () => {
  isVisible.value = !isVisible.value
}
</script>

<!-- ComponentB.vue -->
<script setup lang="ts">
const isVisible = ref(false)
const toggle = () => {
  isVisible.value = !isVisible.value
}
</script>
```

### 抽出後（共通化）

```typescript
// composables/useToggle.ts

/**
 * 真偽値のトグル状態を管理する汎用 Composable
 * @param initialValue - 初期値（デフォルト: false）
 */
export const useToggle = (initialValue = false) => {
  const isActive = ref(initialValue)

  const toggle = (): void => {
    isActive.value = !isActive.value
  }

  const activate = (): void => {
    isActive.value = true
  }

  const deactivate = (): void => {
    isActive.value = false
  }

  return {
    isActive: readonly(isActive),
    toggle,
    activate,
    deactivate,
  }
}
```

---

## パターン 3: 依存方向の正規化

依存が逆方向になっている場合に正規化する。

```
page → model → ui → composables（正方向のみ OK）
```

### 逆方向依存の例（修正前）

```vue
<!-- components/ui/BaseCard.vue — ui が model を依存している（NG）-->
<script setup lang="ts">
import type { Article } from '@/components/model/Article/ArticleCard.vue'
</script>
```

### 正規化後（型を types/ に移動）

```typescript
// types/models.ts — 共有型はここで定義する
export interface Article {
  id: number
  title: string
  publishedAt: string
}
```

```vue
<!-- components/ui/BaseCard.vue — ui は types/ にのみ依存する（OK）-->
<script setup lang="ts">
import type { Article } from '@/types/models'
</script>
```

---

## パターン 4: 肥大化コンポーネントの分割

1 コンポーネントが複数の責務を持っている場合に分割する。

### 分割の判断基準

- コンポーネントが 200 行を超えている
- 1 ファイルに複数の独立した UI 区画がある
- ロジックとテンプレートが混在して読みにくい

### 分割後の構成例

```
components/page/Company/
├── CompanyPageWrapper.page.vue     # レイアウト・Suspense のみ
├── CompanyHero.vue                 # ヒーローセクション
├── CompanyOverview.vue             # 会社概要セクション
└── CompanyAccess.vue               # アクセスセクション
```

---

## パターン 5: utils への純粋関数抽出

副作用のない計算ロジックは `utils/` に移動する。

### 抽出前

```vue
<script setup lang="ts">
// コンポーネント内に計算ロジックが埋め込まれている
const formatDate = (dateStr: string) => {
  const d = new Date(dateStr)
  return `${d.getFullYear()}年${d.getMonth() + 1}月${d.getDate()}日`
}
</script>
```

### 抽出後

```typescript
// utils/helpers.ts

/**
 * ISO 日付文字列を日本語表記に変換する
 * 例: "2025-01-15" → "2025年1月15日"
 * @param dateStr - ISO 8601 形式の日付文字列
 */
export const formatJapaneseDate = (dateStr: string): string => {
  const d = new Date(dateStr)
  return `${d.getFullYear()}年${d.getMonth() + 1}月${d.getDate()}日`
}
```
