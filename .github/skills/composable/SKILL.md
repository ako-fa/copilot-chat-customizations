---
name: composable
description: "Vue 3 Composable 設計の再利用可能なコードパターン集"
---

# Composable Skill

Vue 3 Composition API における Composable 設計の再利用可能なコードパターン。

## 基本原則

- Composable 名は必ず `use` プレフィックスで命名する
- 1 composable = 1 責務を厳守し、関心ごとを混在させない
- 外部公開する状態は `readonly()` で保護し、戻り値の型を明示する
- 副作用（イベントリスナー、タイマー、アニメーション、RAF）はライフサイクルフックで管理する
- `onUnmounted` で必ず cleanup を実装し、登録済みリソースを解除する
- ブラウザ API を扱う場合は SSR を考慮してガードする

## 基本 Composable パターン

```typescript
// composables/useCounter.ts

/**
 * カウンター機能を提供する Composable
 *
 * @param initialValue - カウンターの初期値
 * @returns count（readonly）, increment, decrement, reset
 */
export const useCounter = (
  initialValue = 0,
): {
  count: Readonly<Ref<number>>;
  increment: () => void;
  decrement: () => void;
  reset: () => void;
} => {
  const count = ref(initialValue);

  const increment = (): void => {
    count.value++;
  };
  const decrement = (): void => {
    count.value--;
  };
  const reset = (): void => {
    count.value = initialValue;
  };

  return {
    count: readonly(count), // readonly で外部公開
    increment,
    decrement,
    reset,
  };
};
```

## ライフサイクル管理パターン

イベントリスナーの登録・解除を `onMounted`/`onUnmounted` で管理する：

```typescript
// composables/useKeyBind.ts

/**
 * キーボード操作の監視を提供する Composable
 *
 * @param keys - 監視するキーの配列
 * @param callback - キー一致時に実行するコールバック
 * @returns pressedKeys（readonly）
 */
export const useKeyBind = (
  keys: string[],
  callback?: () => void,
): { pressedKeys: Readonly<Ref<Set<string>>> } => {
  const pressedKeys = ref<Set<string>>(new Set());

  const handleKeyDown = (event: KeyboardEvent): void => {
    const key = event.key.toLowerCase();
    const ctrlKey = event.ctrlKey || event.metaKey;

    const isMatch = keys.some((k) => {
      if (k === "ctrl+s" && ctrlKey && key === "s") return true;
      if (k === key) return true;
      return false;
    });

    if (isMatch) {
      event.preventDefault();
      pressedKeys.value.add(key);
      callback?.();
    }
  };

  const handleKeyUp = (): void => {
    pressedKeys.value = new Set();
  };

  // ライフサイクルでの登録・解除
  onMounted(() => {
    window.addEventListener("keydown", handleKeyDown);
    window.addEventListener("keyup", handleKeyUp);
  });

  onUnmounted(() => {
    window.removeEventListener("keydown", handleKeyDown);
    window.removeEventListener("keyup", handleKeyUp);
  });

  return { pressedKeys: readonly(pressedKeys) };
};
```

## 非同期データ取得パターン

```typescript
// composables/useFetchData.ts

/**
 * API からのデータ取得を管理する Composable
 *
 * @param url - 取得先 URL
 * @returns data（readonly）, loading（readonly）, error（readonly）, refetch
 */
export const useFetchData = <T>(
  url: string,
): {
  data: Readonly<Ref<T | null>>;
  loading: Readonly<Ref<boolean>>;
  error: Readonly<Ref<Error | null>>;
  refetch: () => Promise<void>;
} => {
  const data = ref<T | null>(null) as Ref<T | null>;
  const loading = ref(false);
  const error = ref<Error | null>(null);

  const fetch = async (): Promise<void> => {
    loading.value = true;
    error.value = null;
    try {
      const response = await $fetch<T>(url);
      data.value = response;
    } catch (e) {
      error.value = e as Error;
    } finally {
      loading.value = false;
    }
  };

  onMounted(() => {
    fetch();
  });

  return {
    data: readonly(data),
    loading: readonly(loading),
    error: readonly(error),
    refetch: fetch,
  };
};
```

## スクロール検知 + @vueuse/core 活用パターン

```typescript
// composables/useScrollAwareAnimation.ts
import { useWindowScroll, useRafFn } from "@vueuse/core";

/**
 * スクロール位置に基づくアニメーション値を提供する Composable
 *
 * @returns scrollY の readonly ref
 */
export const useScrollAwareAnimation = (): {
  scrollY: Readonly<Ref<number>>;
} => {
  const { y: scrollY } = useWindowScroll();

  const { pause, resume } = useRafFn(() => {
    // スクロール位置を使用してアニメーション値を計算
    const progress = Math.min(scrollY.value / window.innerHeight, 1);
    // progress を使用した処理
  });

  onMounted(resume);
  onUnmounted(pause);

  return { scrollY };
};
```

## Composable 設計チェック項目

1. **`use*` プレフィックス**: 必ず `use` で始める（自動インポート対象）
2. **返り値の `readonly()`**: 外部に公開する ref 値は `readonly()` で保護する
3. **クリーンアップ**: `onUnmounted` でイベントリスナー・タイマー・アニメーションを解除する
4. **型注釈**: パラメータと返り値に明示的な型を付ける
5. **TSDoc**: 目的・パラメータ・返り値を日本語で記述する
6. **SSR 考慮**: ブラウザ API を使う場合は `import.meta.server` でガードする

## functional コンポーネント → Composable 変換ルール

見た目を伴わない機能コンポーネントは Composable に変換する：

```vue
<!-- ❌ 非推奨: 見た目なしのコンポーネント -->
<template>
  <slot />
</template>

<script setup lang="ts">
// ロジックのみ、テンプレートは slot を通すだけ
</script>
```

↓ 変換

```typescript
// ✅ 推奨: Composable として実装
export const useFeature = () => {
  // ロジックを直接実装
  // use* プリフィックスで自動インポート
  // ライフサイクル管理が明示的
};
```

## 実施手順

1. 要件から責務を特定し、composable の粒度を決定する
2. 戻り値の型定義を設計する
3. `use` プレフィックスで命名し、composable を実装する
4. ライフサイクルフックによる cleanup 処理を実装する
5. 呼び出し元コンポーネントで動作を確認する

## チェックリスト

- `use` プレフィックスで命名されているか
- 1 composable = 1 責務になっているか
- 戻り値の ref が必要に応じて `readonly()` で保護されているか
- パラメータと戻り値の型が明示されているか
- 副作用が `onMounted` / `onUnmounted` などのフックで管理されているか
- `onUnmounted` でイベントリスナー、タイマー、アニメーション、RAF の cleanup が行われているか
- TSDoc が日本語で記述されているか
- ブラウザ API 利用時に SSR ガードがあるか
