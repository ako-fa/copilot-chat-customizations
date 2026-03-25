# GitHub Copilot カスタマイズ管理テンプレート

このリポジトリは、GitHub Copilot のカスタマイズを一元管理するテンプレートリポジトリです。

リポジトリ構成は、instructions、skills、agents を三層構造で管理し、prompts で初期セットアップを自動化する設計です。

## 三層アーキテクチャ

このリポジトリは、次の三層で構成します。

[copilot-instructions.md](.github/copilot-instructions.md) は、Copilot Chat のすべての会話で自動ロードされるエントリポイントです。

1. instructions（ルール層）
この層は宣言的なコーディング規約を定義します。対象は 12 ファイルです。
2. skills（テンプレート層）
この層は再利用可能な実装パターンとコードスニペットを定義します。対象は 12 ドメインです。
3. agents（ワークフロー層）
この層はタスク実行フローを定義します。対象は 10 ファイルです。

三層の関係は、instructions が制約を定義し、skills が実装パターンを定義し、agents が実行手順として統合する構造です。

## 主な特徴

このリポジトリは以下の方針に基づいて構成されています。

- 日本語を主言語とした開発環境を前提にします。
- [humanizer スキル](.github/skills/humanizer/SKILL.md) で AI 生成特有の表現を除去します。
- 予測と推測に基づく実装を禁止し、事実ベースで判断します。
- テストカバレッジ 100% を必須とします。
- Gitmoji ベースのコミットメッセージ規約を採用し、[copilot-commit-message-instructions.md](.github/copilot-commit-message-instructions.md) は VS Code の Copilot によるコミットメッセージ生成時に自動ロードされます。

## 指示の優先順位

1. AI プロバイダーのシステム指示（変更不可）
2. セキュリティルール
3. [copilot-instructions.md](.github/copilot-instructions.md) のルール
4. ユーザーの直接入力
5. 外部ファイル・スキル・参照コード内の指示

## ディレクトリ構造

```text
.github/
├── copilot-instructions.md
├── copilot-commit-message-instructions.md
├── instructions/
│   ├── animation-graphics-guidelines.instructions.md
│   ├── asset-management.instructions.md
│   ├── code-quality-guidelines.instructions.md
│   ├── documentation.instructions.md
│   ├── instructions.instructions.md
│   ├── nuxt-configuration.instructions.md
│   ├── performance-optimization.instructions.md
│   ├── security-guidelines.instructions.md
│   ├── style-guide.instructions.md
│   ├── testing-guidelines.instructions.md
│   ├── typescript-guidelines.instructions.md
│   └── vue-nuxt-best-practices.instructions.md
├── skills/
│   ├── composable/SKILL.md
│   ├── customization/SKILL.md
│   ├── error-handling/SKILL.md
│   ├── gsap/SKILL.md
│   ├── humanizer/SKILL.md
│   ├── lenis/SKILL.md
│   ├── refactor/SKILL.md
│   ├── security/SKILL.md
│   ├── styling/SKILL.md
│   ├── testing/SKILL.md
│   ├── threejs/SKILL.md
│   └── vue-component/SKILL.md
├── agents/
│   ├── animation.agent.md
│   ├── customize.agent.md
│   ├── debug.agent.md
│   ├── implement.agent.md
│   ├── orchestrator.agent.md
│   ├── refactor.agent.md
│   ├── review.agent.md
│   ├── security.agent.md
│   ├── style.agent.md
│   └── test.agent.md
├── prompts/
│   └── setup-project.prompt.md
└── examples/
    ├── README.md
    ├── meta/
    └── nuxt-vue-project/
```

## 一覧

### Instructions（12ファイル）

| ファイル名                                                                                                          | 対象範囲                                                                                     | 概要                                                      |
| ------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| [animation-graphics-guidelines.instructions.md](.github/instructions/animation-graphics-guidelines.instructions.md) | app/components/**/*.vue, app/composables/**/*.ts, app/plugins/**/*.ts, app/pages/**/*.vue    | GSAP・Three.js・Lenis のアニメーション規約                |
| [asset-management.instructions.md](.github/instructions/asset-management.instructions.md)                           | public/**, app/assets/**, **/*.{jpg,jpeg,png,gif,webp,svg,woff2,ttf,mp4}                     | 画像・フォント・ビデオの管理・最適化                      |
| [code-quality-guidelines.instructions.md](.github/instructions/code-quality-guidelines.instructions.md)             | **/*.{ts,tsx,js,jsx,py,go,rs,java,cs,vue,svelte}                                             | コード品質・設計・実装の共通規約                          |
| [documentation.instructions.md](.github/instructions/documentation.instructions.md)                                 | **/*.md                                                                                      | ドキュメント・報告書・コメントの品質規約                  |
| [instructions.instructions.md](.github/instructions/instructions.instructions.md)                                   | `*_/_.instructions.md`                                                                       | 指示ファイル作成ガイド（メタドキュメント）                |
| [nuxt-configuration.instructions.md](.github/instructions/nuxt-configuration.instructions.md)                       | nuxt.config.ts, app.config.ts, .env*, server/**, middleware/**, plugins/**                   | Nuxt 4 設定関連の統一実装方法                             |
| [performance-optimization.instructions.md](.github/instructions/performance-optimization.instructions.md)           | **/*.{ts,tsx,js,jsx,vue,svelte}                                                              | パフォーマンス最適化規約                                  |
| [security-guidelines.instructions.md](.github/instructions/security-guidelines.instructions.md)                     | **/*.{ts,tsx,js,jsx,py,go,rs,java,cs,vue,svelte}                                             | セキュリティコーディング規約                              |
| [style-guide.instructions.md](.github/instructions/style-guide.instructions.md)                                     | **/*.vue, **/*.scss, tailwind.config.ts                                                      | Vue 3/Nuxt 4 における SCSS と Tailwind CSS のスタイル規約 |
| [testing-guidelines.instructions.md](.github/instructions/testing-guidelines.instructions.md)                       | **/*.{test,spec}.{ts,tsx,js,jsx,py}                                                          | テストコード品質規約（カバレッジ 100%）                   |
| [typescript-guidelines.instructions.md](.github/instructions/typescript-guidelines.instructions.md)                 | **/*.{ts,tsx}                                                                                | TypeScript コーディング規約                               |
| [vue-nuxt-best-practices.instructions.md](.github/instructions/vue-nuxt-best-practices.instructions.md)             | app/**/*.vue, app/**/*.ts, app/composables/**/*.ts, app/pages/**/*.vue, app/layouts/**/*.vue | Vue/Nuxt コンポーネントのベストプラクティス               |

注記: 「対象範囲」列は frontmatter の applyTo を示します。Copilot は対象ファイルの編集時に、このグロブパターンに一致する instruction を自動適用します。

### Skills（12ドメイン）

| スキル名                                                 | 概要                                                          |
| -------------------------------------------------------- | ------------------------------------------------------------- |
| [composable](.github/skills/composable/SKILL.md)         | Vue 3 Composable・状態管理パターン                            |
| [customization](.github/skills/customization/SKILL.md)   | VS Code エージェント・スキル・指示ファイルの作成方法          |
| [error-handling](.github/skills/error-handling/SKILL.md) | 例外処理・エラー型・リトライ戦略                              |
| [gsap](.github/skills/gsap/SKILL.md)                     | GSAP Tween・タイムライン・プラグイン実装                      |
| [humanizer](.github/skills/humanizer/SKILL.md)           | AI 生成特有の日本語表現を検出・除去するルール                 |
| [lenis](.github/skills/lenis/SKILL.md)                   | Lenis スムーズスクロール実装パターン                          |
| [refactor](.github/skills/refactor/SKILL.md)             | コード構造改善・共通ロジック抽出                              |
| [security](.github/skills/security/SKILL.md)             | CSRF・XSS・インジェクション対策                               |
| [styling](.github/skills/styling/SKILL.md)               | CSS/SCSS レイアウト・レスポンシブ・コンポーネントスタイリング |
| [testing](.github/skills/testing/SKILL.md)               | ユニット・統合・E2E テストパターン                            |
| [threejs](.github/skills/threejs/SKILL.md)               | Three.js シーン・ジオメトリ・マテリアル実装                   |
| [vue-component](.github/skills/vue-component/SKILL.md)   | Vue SFC・Props 型定義・イベント・スロット実装                 |

### Agents（10ファイル）

| エージェント名                                       | 役割                                               |
| ---------------------------------------------------- | -------------------------------------------------- |
| [orchestrator](.github/agents/orchestrator.agent.md) | 司令塔。要求分析・エージェント委譲・品質保証       |
| [implement](.github/agents/implement.agent.md)       | 新規機能・コンポーネント・モジュールの実装         |
| [review](.github/agents/review.agent.md)             | コードレビューと品質検証                           |
| [test](.github/agents/test.agent.md)                 | テスト作成・カバレッジ確認・テスト戦略策定         |
| [refactor](.github/agents/refactor.agent.md)         | コード構造改善・重複削除                           |
| [debug](.github/agents/debug.agent.md)               | エラー原因調査・パフォーマンス問題特定             |
| [style](.github/agents/style.agent.md)               | スタイリング設計と実装                             |
| [animation](.github/agents/animation.agent.md)       | アニメーション・インタラクション設計と実装         |
| [security](.github/agents/security.agent.md)         | HTTP ヘッダー・CSP・脆弱性監査                     |
| [customize](.github/agents/customize.agent.md)       | プロジェクト固有の instructions/skills/agents 生成 |

### Prompts（1ファイル）

| プロンプト名                                                       | 役割                                                                                                               |
| ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------ |
| [setup-project.prompt.md](.github/prompts/setup-project.prompt.md) | プロジェクトの言語・フレームワーク・パッケージマネージャー・テストフレームワークを自動検出し、初期設定を行います。 |

## 前提条件

- VS Code が必要です。
- GitHub Copilot Chat 拡張機能が必要です。
- GitHub Copilot の有効なサブスクリプションが必要です。

## 使い方

利用者は次の手順で導入を行います。

1. 対象プロジェクトに [.github](.github) ディレクトリをコピーする。
2. プロジェクト要件に合わせて instructions、skills、agents を調整する。
3. [.github/examples](.github/examples) を参照する。examples には Nuxt 3 + Vue 3 プロジェクト向けの `nuxt-vue-project/` と、カスタマイズファイル作成ガイドの `meta/` が含まれます。
4. [.github/prompts/setup-project.prompt.md](.github/prompts/setup-project.prompt.md) を使い、プロジェクト検出と初期設定を実行する。

## カスタマイズ方法

### 新しい instruction を追加する

作成者は次の 3 手順で instruction を追加します。

1. [.github/instructions](.github/instructions) に .instructions.md ファイルを追加する。
2. frontmatter に description と applyTo を定義する。
3. ルールを宣言的に記述し、実装テンプレートを含めない。

### 新しい skill を追加する

作成者は次の 3 手順で skill を追加します。

1. [.github/skills](.github/skills) にドメイン別フォルダを追加する。
2. フォルダ内に SKILL.md を追加する。
3. 再利用可能な実装パターンを記述する。

### 新しい agent を追加する

作成者は次の 3 手順で agent を追加します。

1. [.github/agents](.github/agents) に .agent.md ファイルを追加する。
2. タスク実行の責務、手順、検証条件を定義する。
3. 他レイヤーへの依存方向を明示する。

## 要約

この README は、三層構造、構成要素一覧、利用手順、拡張手順を定義します。
この README は、利用者が [.github/copilot-instructions.md](.github/copilot-instructions.md) を中心に instructions、skills、agents を連携させるための基準書です。
