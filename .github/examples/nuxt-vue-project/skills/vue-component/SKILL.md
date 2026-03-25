---
description: "Vue 3 コンポーネント設計の再利用可能なコードパターン集"
---

# Vue Component Skill

Vue 3 + Nuxt 4 におけるコンポーネント設計の再利用可能なコードパターン。

## SFC 標準構造

```vue
<script setup lang="ts">
// 1. 型定義
interface Props {
  title: string;
  count?: number;
  disabled?: boolean;
}

// 2. Props 定義（すべてに型・必須/デフォルトを与える）
const props = withDefaults(defineProps<Props>(), {
  count: 0,
  disabled: false,
});

// 3. Emits 定義
const emit = defineEmits<{
  "update:modelValue": [value: string];
  close: [];
}>();

// 4. リアクティブな状態
const isOpen = ref(false);

// 5. 計算プロパティ
const displayText = computed(() => props.title.toUpperCase());

// 6. メソッド
const handleClick = (): void => {
  emit("update:modelValue", "new-value");
};

// 7. ライフサイクル
onMounted(() => {
  /* 初期化処理 */
});
onUnmounted(() => {
  /* クリーンアップ処理 */
});
</script>

<template>
  <div class="component-root">
    <h2>{{ title }}</h2>
    <button @click="handleClick">クリック</button>
  </div>
</template>

<style scoped lang="scss">
.component-root {
  // スタイル
}
</style>
```

## ページラッパーパターン

```vue
<!-- pages/example.vue: ルーティング定義のみ -->
<script setup lang="ts">
definePageMeta({ layout: "default" });
</script>

<template>
  <ExamplePageWrapper />
</template>
```

```vue
<!-- components/page/Example/ExamplePageWrapper.page.vue: レイアウト -->
<template>
  <div class="page-wrapper">
    <Suspense>
      <template #default>
        <ExampleContent />
      </template>
    </Suspense>
  </div>
</template>
```

```vue
<!-- components/page/Example/ExampleContent.vue: 非同期処理 -->
<script setup lang="ts">
const { data } = await useFetch("/api/example");
</script>

<template>
  <main><!-- ページコンテンツ --></main>
</template>
```

## Props 設計パターン

```typescript
// Props は不変として扱う。直接変更しない
const props = defineProps<{ count: number }>();

// ✅ 変更は emit 経由で親に委譲する
const increment = (): void => {
  emit("increment", props.count + 1);
};

// ❌ 禁止: Props の直接変更
// props.count++
```

## v-model 双方向バインディング

```vue
<!-- カスタム v-model コンポーネント -->
<template>
  <input
    :value="modelValue"
    @input="
      emit('update:modelValue', ($event.target as HTMLInputElement).value)
    "
  />
</template>

<script setup lang="ts">
defineProps<{ modelValue: string }>();
const emit = defineEmits<{ "update:modelValue": [value: string] }>();
</script>
```

## 親子通信パターン

```vue
<!-- 親コンポーネント -->
<template>
  <ChildComponent :count="count" @update:count="count = $event" />
</template>

<script setup lang="ts">
const count = ref(0);
</script>
```

```vue
<!-- 子コンポーネント -->
<script setup lang="ts">
interface Props {
  count: number;
}
const props = defineProps<Props>();
const emit = defineEmits<{ "update:count": [value: number] }>();

const handleIncrement = (): void => {
  emit("update:count", props.count + 1);
};
</script>

<template>
  <button @click="handleIncrement">{{ count }}</button>
</template>
```

## テンプレートルール

- `v-for` には常に安定した `key` を付与する
- 同一要素で `v-if` と `v-for` を併用しない（computed で事前フィルタする）
- コンポーネント名は SFC では PascalCase、DOM テンプレートでは kebab-case
- テンプレート式を複雑にしない（computed/method に移す）
- `v-html` はサニタイズ済みかつ信頼できる文字列に限定し、コメントで意図を明記する

## 命名規則

| 要素                | 規則                     | 例                                 |
| ------------------- | ------------------------ | ---------------------------------- |
| コンポーネント      | PascalCase、マルチワード | `BaseButton.vue`, `UserAvatar.vue` |
| ページラッパー      | PascalCase + `.page.vue` | `TopPageWrapper.page.vue`          |
| Composable          | camelCase + `use` prefix | `useCounter.ts`, `useKeyBind.ts`   |
| ユーティリティ      | camelCase                | `formatDate.ts`, `helpers.ts`      |
| 定数                | SCREAMING_SNAKE_CASE     | `API_ENDPOINTS`, `MAX_RETRY_COUNT` |
| 型/インターフェース | PascalCase               | `UserProfile`, `ApiResponse`       |

## コンポーネント接頭辞

- `Base*`: 再利用可能な基本 UI 要素（BaseButton, BaseCard）
- `App*`: アプリケーション全体で使う単一例（AppHeader, AppFooter）
- `The*`: ページ固有の親レベル（TheHeroSection, TheSidebar）
