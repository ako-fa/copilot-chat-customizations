---
description: 'Vitest テスト実装の再利用可能なコードパターン集'
---

# Testing Skill

Vitest 4.0.18 を使用したテスト実装の再利用可能なコードパターン。

## テストファイル構造テンプレート

```typescript
/**
 * [対象名] のテスト
 *
 * 責務:
 * - [テスト対象の責務を箇条書き]
 *
 * テスト観点:
 * - [正常系のテストケース]
 * - [異常系のテストケース]
 * - [境界値のテストケース]
 */

import { beforeEach, describe, expect, it, vi } from 'vitest'

// モックは最上位で定義（vi.mock は巻き上げられる）
vi.mock('@/composables/useSomeComposable', () => ({
  useSomeComposable: vi.fn(() => ({ value: 'mocked' })),
}))

describe('[対象名]', () => {
  beforeEach(() => {
    vi.clearAllMocks() // 各テスト前にモックをリセット
  })

  describe('正常系', () => {
    it('期待する動作の日本語説明', () => {
      // 前提条件: テストデータの準備
      const input = '芝野太郎'

      // 実行: 対象を呼び出し
      const result = targetFunction(input)

      // 期待: 結果を検証
      expect(result).toBe('期待値')
    })
  })
})
```

## Composable テストパターン

```typescript
/**
 * useSomeFeature Composable テスト
 *
 * 責務:
 * - [Composable の主要な責務]
 *
 * テスト観点:
 * - API 公開: 公開される関数/プロパティが正しく提供される
 * - 初期化: 正常に初期化され、エラーが発生しない
 * - エラーハンドリング: 異常時に適切にエラーをスローする
 */

import { useSomeFeature } from '@/composables/useSomeFeature'
import { beforeEach, describe, expect, it, vi } from 'vitest'
import { ref } from 'vue'

vi.mock('#app', () => ({
  useNuxtApp: vi.fn(() => ({
    $plugin: mockPlugin,
  })),
}))

describe('useSomeFeature', () => {
  beforeEach(() => {
    vi.clearAllMocks()
  })

  describe('初期化', () => {
    it('正常に初期化される', () => {
      // 前提条件: 必要なパラメータを準備
      const target = ref<HTMLElement | null>(document.createElement('div'))

      // 実行: Composable を呼び出し
      const result = useSomeFeature(target)

      // 期待: 公開 API が返される
      expect(result).toBeDefined()
      expect(typeof result.someMethod).toBe('function')
    })
  })

  describe('プラグイン初期化チェック', () => {
    it('プラグインが初期化されていない場合、エラーをスロー', () => {
      // 前提条件: プラグインがない状態を設定
      vi.mocked(useNuxtApp).mockReturnValue({
        $plugin: undefined,
      } as TestNuxtApp)

      const target = ref<HTMLElement | null>(null)

      // 実行 & 期待: エラーをスローする
      expect(() => useSomeFeature(target)).toThrow('Plugin not initialized')
    })
  })
})
```

## Vue コンポーネントテストパターン

```typescript
import { mount } from '@vue/test-utils'
import { describe, expect, it } from 'vitest'
import BaseButton from '@/components/ui/BaseButton/BaseButton.vue'

describe('BaseButton', () => {
  describe('Props の表示', () => {
    it('label prop が正しく表示される', () => {
      // 前提条件: label を指定してマウント
      const wrapper = mount(BaseButton, {
        props: { label: '送信' },
      })

      // 期待: ラベルが DOM に表示される
      expect(wrapper.text()).toContain('送信')
    })
  })

  describe('イベントの発火', () => {
    it('クリック時に click イベントが発火する', async () => {
      // 前提条件: コンポーネントをマウント
      const wrapper = mount(BaseButton, {
        props: { label: '送信' },
      })

      // 実行: ボタンをクリック
      await wrapper.trigger('click')

      // 期待: click イベントが 1 回発火
      expect(wrapper.emitted('click')).toHaveLength(1)
    })
  })
})
```

## テーブルドリブンテスト（境界値）

```typescript
describe('境界値テスト', () => {
  const testCases = [
    { 入力: '', 期待: '必須エラー', 説明: '空文字列' },
    { 入力: 'abc', 期待: 'OK', 説明: '正常値' },
    { 入力: 'あ'.repeat(201), 期待: '長さ超過', 説明: '最大文字数超過' },
  ]

  testCases.forEach(({ 入力, 期待, 説明 }) => {
    it(`${説明}（入力: "${入力}"）の場合、${期待}`, () => {
      // 実行: バリデーション実行
      const result = validateName(入力)

      // 期待
      expect(result).toBe(期待)
    })
  })
})
```

## Nuxt コンポーネントスタブ化

```typescript
const wrapper = mount(Navigation, {
  global: {
    stubs: {
      NuxtLink: {
        template: '<a :to="to"><slot /></a>',
        props: ['to'],
      },
      NuxtImg: {
        template: '<img :src="src" :alt="alt" />',
        props: ['src', 'alt', 'format'],
      },
    },
  },
})
```

## 非同期処理テスト

```typescript
import { flushPromises } from '@vue/test-utils'

it('データ取得後に表示が更新される', async () => {
  // 前提条件: モックデータを準備
  const mockData = { id: 1, name: 'テストユーザー' }
  vi.mocked(useFetch).mockResolvedValue(mockData)

  // 実行: コンポーネントをマウント
  const wrapper = mount(UserProfile)
  await flushPromises() // すべての Promise が解決されるまで待つ

  // 期待: 取得したデータが表示される
  expect(wrapper.text()).toContain('テストユーザー')
})
```

## 実装詳細に依存しないテスト

```typescript
// ❌ 悪い例: 内部メソッドに依存
it('内部メソッドの動作', () => {
  const wrapper = mount(Component)
  ;(wrapper.vm as any).internalMethod() // 実装詳細に依存
})

// ✅ 良い例: ユーザー行動で検証
it('ボタンクリック時の動作', async () => {
  // 前提条件: コンポーネントをマウント
  const wrapper = mount(Component)

  // 実行: ユーザー行動をシミュレート
  await wrapper.find('button').trigger('click')

  // 期待: 表示結果で検証
  expect(wrapper.text()).toContain('クリック済み')
})
```

## テスト実装ルール

| ルール                   | 内容                                            |
| ------------------------ | ----------------------------------------------- |
| **ファイル名**           | `*.test.ts`                                     |
| **テスト名**             | 日本語で記述する                                |
| **AAA パターン**         | 前提条件 → 実行 → 期待 のコメントを必ず付ける   |
| **テストデータ**         | 意味が即座に明確になる日本語文字列を使用する    |
| **Nuxt コンポーネント**  | NuxtLink, NuxtImg はスタブ化する                |
| **モックリセット**       | `beforeEach` で `vi.clearAllMocks()` を実行する |
| **実装詳細への依存禁止** | `(wrapper.vm as any)` のようなアクセスは禁止    |

## テスト実行コマンド

```bash
pnpm test                       # 全テスト実行
pnpm test:watch                 # ウォッチモード
pnpm test:coverage              # カバレッジ計測
pnpm test path/to/test.test.ts  # 特定ファイル（-- を挟まない）
```
