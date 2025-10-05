# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Language Preference

**IMPORTANT: Always respond in Japanese (日本語) when working in this repository.**

このコードベースは日本語が主言語です。UI テキスト、コメント、ドキュメントは日本語で記述されています。開発者とのすべてのやり取りは、一貫性と明瞭性を保つために日本語で行ってください。

---

## Overview

This is **GeminiPWA (Vue Edition)** - a complete re-architecture of the original GeminiPWA using Vue.js 3. It is a Progressive Web Application (PWA) for chatting with multiple AI APIs including Gemini, DeepSeek, Claude (Anthropic), OpenAI, xAI (Grok), and LLM aggregators like OpenRouter.

**これは Vue.js 3 を使用して再構築された GeminiPWA（Vue 版）です。** 既存の機能を完全に踏襲しつつ、メンテナンス性とパフォーマンスを大幅に向上させています。

### Key Characteristics

- **Vue.js 3 フレームワーク**: Composition API を使用した最新のアーキテクチャ
- **完全モジュール化**: 機能ごとに細かくパッケージング（100+ ファイル構成）
- **パフォーマンス最適化**: スマートフォンでの利用をメインに最適化
- **ビルドプロセス**: Vite を使用した高速ビルド
- **クライアントサイド実行**: サーバーサイド処理不要（静的ホスティングのみ）
- **IndexedDB**: 全データをブラウザ内にローカル保存
- **日本語 UI**: 日本語を主言語とした UI

---

## Core Architecture

### Technology Stack

| カテゴリ | 技術 | 用途 |
|---------|------|------|
| **フレームワーク** | Vue.js 3.4+ | リアクティブ UI |
| **状態管理** | Pinia 2.1+ | グローバル状態管理 |
| **ルーティング** | Vue Router 4.2+ | SPA ルーティング |
| **ビルドツール** | Vite 5.0+ | 高速ビルド・開発サーバー |
| **仮想スクロール** | vue-virtual-scroller | 大量メッセージ対応 |
| **Markdown** | markdown-it 14.0+ | Markdown 解析 |
| **サニタイザ** | DOMPurify 3.0+ | XSS 防止 |
| **IndexedDB** | idb 8.0+ | データベース操作 |
| **PWA** | vite-plugin-pwa | Service Worker 自動生成 |

### Project Structure

```
src/
├── main.js                 # エントリーポイント
├── App.vue                 # ルートコンポーネント
├── router/                 # Vue Router 設定
├── stores/                 # Pinia 状態管理
│   ├── chat.js             # チャット状態
│   ├── session.js          # セッション管理
│   ├── settings.js         # 設定管理
│   ├── apiProvider.js      # API プロバイダー状態
│   ├── twinEngine.js       # Twin-Engine 状態
│   └── ui.js               # UI 状態
├── views/                  # ページコンポーネント
│   ├── ChatView.vue        # チャット画面
│   ├── HistoryView.vue     # 履歴一覧画面
│   └── SettingsView.vue    # 設定画面
├── components/             # 再利用可能コンポーネント
│   ├── chat/               # チャット関連
│   ├── session/            # セッション関連
│   ├── settings/           # 設定関連
│   ├── common/             # 共通コンポーネント
│   └── features/           # 高度な機能
├── services/               # ビジネスロジック層
│   ├── api/                # API 通信サービス
│   │   ├── gemini.js
│   │   ├── deepseek.js
│   │   ├── claude.js
│   │   ├── openai.js
│   │   ├── xai.js
│   │   ├── llmaggregator.js
│   │   └── dummy.js
│   ├── db/                 # IndexedDB 操作
│   ├── features/           # 高度な機能（Twin-Engine等）
│   └── export/             # エクスポート・インポート
├── composables/            # Vue Composables
├── utils/                  # ユーティリティ関数
├── workers/                # Web Workers
├── config/                 # 設定ファイル
│   ├── defaults.js         # デフォルト設定値
│   ├── themes.js           # テーマ定義
│   ├── providers.js        # API プロバイダー定義
│   └── models.js           # モデルリスト
└── assets/                 # 静的リソース
    └── styles/
```

### State Management (Pinia Stores)

**状態管理は Pinia で一元管理されています。**

#### `stores/chat.js` - チャット状態

```javascript
// 管理している状態
{
  currentMessages: [],      // 現在のメッセージ配列
  isSending: false,         // 送信中フラグ
  isSummarizing: false,     // 要約中フラグ
  isProofreading: false,    // 校正中フラグ
  abortController: null,    // API 中断用
  streamingText: '',        // ストリーミング中テキスト
  messageCollapsedStates: Map, // メッセージ折りたたみ状態
  thoughtStates: Map        // 思考プロセス開閉状態
}
```

#### `stores/session.js` - セッション管理

```javascript
{
  currentSessionId: null,   // 現在のセッション ID
  sessions: [],             // 全セッション一覧
  linkedSessionIds: [],     // リンクされたセッション ID
  sortOrder: 'updatedAt'    // ソート順
}
```

#### `stores/settings.js` - 設定管理

```javascript
{
  theme: 'light',           // テーマ
  apiProvider: 'gemini',    // 使用プロバイダー
  apiKeys: {},              // API キー（プロバイダーごと）
  systemPrompts: {},        // システムプロンプト
  parameters: {},           // API パラメータ
  ui: {},                   // UI 設定
  advanced: {}              // 高度な設定
}
```

### Routing

```javascript
// router/index.js
{
  '/': ChatView,               // チャット画面（デフォルト）
  '/chat/:sessionId': ChatView, // セッション指定チャット
  '/history': HistoryView,     // 履歴一覧
  '/settings': SettingsView    // 設定画面
}
```

---

## Key Features

### すべての既存機能を踏襲

このアプリケーションは、既存の GeminiPWA（Vanilla JS 版）の全機能を引き継いでいます。

#### 1. セッション管理
- 複数チャットセッションの独立管理
- セッションリンク（「AI 間会話モード」）- 2 つのセッションをリンクして AI 同士の会話を実現
- セッションのインポート/エクスポート（JSON 形式）
- 一括操作（全エクスポート、全インポート、全削除）
- スワイプナビゲーション（左右スワイプでセッション切替）

#### 2. Twin-Engine（実験的機能）
- デュアル API アーキテクチャ：会話用と要約用で異なる API を使用
- 会話履歴の自動または手動要約
- 要約を送信することでトークン使用量を削減
- 要約開始のターン数閾値を設定可能

#### 3. 校正機能（「校正機能」）
- テキスト置換/訂正機能
- AI 応答中の自動テキスト置換に使用可能
- 設定 → 高度なオプションで有効化

#### 4. 高度な UI カスタマイズ
- 豊富なテーマ（パステル系を含む複数のカラースキーム）
- 吹き出し、ヘッダー/フッター、オーバーレイの透明度制御
- メッセージ、コードブロック、思考プロセス要約のフォントサイズ調整
- 背景画像サポート
- チャットアイコン/アバターシステム（SNS スタイル）
- メッセージの折りたたみ/展開機能

#### 5. メッセージ処理
- Markdown レンダリング（シンタックスハイライト付き）
- Mermaid 図のサポート
- Markdown 内の画像表示
- コードブロックのコピーボタン
- 添付ファイルサポート（ファイルと画像）
- メッセージの編集、再試行、複製
- コピーしたテキストを蓄積するクリップボードスタック

#### 6. 思考プロセスのレンダリング
DeepSeek やその他のモデルが「思考モード」をサポートしており、内部推論が最終応答とは別に表示されます。

#### 7. 対応 API プロバイダー
- Google Gemini API（ファイル解析、Grounding、思考プロセス）
- DeepSeek API（思考プロセス、カスタムエンドポイント）
- Anthropic Claude API（Extended Thinking）
- OpenAI API（GPT-4o 等）
- xAI (Grok) API（Vision、推論努力レベル）
- LLM Aggregator（OpenRouter、Together.ai、Fireworks 等）
- Dummy AI（テスト・デバッグ用、空の応答を返す）

---

## Performance Optimization

**このアプリケーションはスマートフォンでの利用を最優先に最適化されています。**

### 目標指標

| 指標 | 目標値 |
|------|-------|
| 初回表示時間（FCP） | 0.8 秒以下 |
| インタラクティブ時間（TTI） | 1.5 秒以下 |
| 初期バンドルサイズ | 80KB 以下（gzip） |
| メモリ使用量（1000 件表示時） | 30MB 以下 |
| スクロール FPS | 60fps 維持 |
| Lighthouse Score | 95 点以上 |

### 最適化手法

1. **仮想スクロール（Virtual Scrolling）**
   - `vue-virtual-scroller` を使用
   - 画面に表示されているメッセージのみをレンダリング
   - メモリ使用量 90%削減、スクロール速度 10 倍向上

2. **Markdown 解析の最適化**
   - 解析結果をキャッシュ（LRU、上限 1000 件）
   - Web Worker でバックグラウンド処理
   - 再レンダリング時の処理時間 95%削減

3. **画像の遅延読み込み（Lazy Loading）**
   - `loading="lazy"` 属性使用
   - 初期表示速度 50%向上

4. **IndexedDB の最適化**
   - ページネーション対応
   - メッセージを別ストアに分離
   - 読み込み速度 5 倍向上

5. **コード分割（Code Splitting）**
   - API プロバイダーごとに分割
   - 使用するプロバイダーのみ読み込み
   - 初期読み込みサイズ 60%削減

6. **リアクティブの最適化**
   - `shallowRef`, `shallowReactive` を適切に使用
   - 不要な深い監視を回避

7. **Service Worker キャッシュ戦略**
   - 静的リソース：CacheFirst
   - API 通信：NetworkFirst

8. **Brotli 圧縮**
   - Gzip より高圧縮率
   - バンドルサイズ追加 15-20%削減

---

## Development

### Setup

```bash
# 依存関係インストール
pnpm install

# 開発サーバー起動
pnpm dev

# ビルド
pnpm build

# プレビュー
pnpm preview

# Lint
pnpm lint

# フォーマット
pnpm format
```

### Local Development

1. **開発サーバー**: `pnpm dev` で Vite 開発サーバーが起動します（通常 http://localhost:5173）
2. **ホットリロード**: ファイル変更時に自動リロード
3. **Vue DevTools**: ブラウザ拡張機能で状態を確認可能

### Testing

**常に無料モデルで先にテストしてから有料 API を使用してください**（予期しないトークン消費を避けるため）。

推奨無料モデル：
- Gemini Flash（API キー必要）
- OpenRouter の無料モデル
- Dummy AI（空の応答を返す、コストゼロ）

### Building

```bash
pnpm build
```

ビルド成果物は `dist/` ディレクトリに出力されます：
- `dist/index.html`
- `dist/assets/` (JS, CSS)
- `dist/manifest.json`
- `dist/sw.js` (Service Worker)

---

## Important Implementation Notes

### API Security

- **すべての API キーはブラウザ内（IndexedDB）にローカル保存されます**
- サーバーサイドへの保存や送信は一切ありません（各 AI プロバイダーへの送信を除く）
- Content Security Policy で外部ドメインを制限（ホワイトリスト方式）
- XSS 防止のため DOMPurify でサニタイズ

### Streaming Implementation

Server-Sent Events (SSE) を使用したストリーミング実装：
- プロバイダーごとに異なるレスポンス形式を処理
- タイピングエフェクト付きのリアルタイムテキストレンダリング
- エラーリカバリーとリトライロジック
- 「思考プロセス」と「応答」を別々に処理

### Data Persistence

- **IndexedDB ストア**:
  - `sessions`: セッション情報
  - `messages`: メッセージ（セッションから分離）
  - `settings`: 設定
- **LocalStorage**: 使用しません（すべて IndexedDB）
- **セッション復元**: アプリ起動時に自動復元
- **キャッシュ管理**: Service Worker + 設定画面から手動更新可能

### Error Handling

- グローバルエラーハンドラーで自動リカバリー
- 1 分以内に 3 回 JavaScript エラーが発生するとキャッシュクリアして自動リロード
- アプリの健全性を監視する Watchdog システム

### Known Limitations

- **ファイル/画像解析**: Gemini 等のマルチモーダルモデルのみ対応（対応モデルを除く）
- **インターネット検索統合**: なし（Gemini の検索はプロバイダー側機能）
- **Claude API**: Web フェッチング機能は未実装
- **OpenAI API**: 思考モードに相当する内部推論フラグなし
- **DeepSeek**: 多くのパラメータが未対応（README 参照）

---

## Modifying the Code

コードを変更する際のガイドライン：

### 1. コンポーネントの追加

```vue
<!-- src/components/chat/NewComponent.vue -->
<template>
  <div class="new-component">
    <!-- UI -->
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useChatStore } from '@/stores/chat'

const chatStore = useChatStore()
const localState = ref(null)
</script>

<style scoped>
.new-component {
  /* スタイル */
}
</style>
```

### 2. 新しい API プロバイダーの追加

```javascript
// src/services/api/newprovider.js
export default class NewProviderAPI {
  async sendMessage(messages, options) {
    // API 通信実装
  }

  async *streamResponse(response) {
    // ストリーミング実装
  }
}
```

```javascript
// src/config/providers.js に追加
export const PROVIDERS = {
  // ...既存プロバイダー
  newprovider: {
    name: 'New Provider',
    defaultModel: 'model-name',
    color: '#hexcolor'
  }
}
```

### 3. Store の追加

```javascript
// src/stores/newStore.js
import { defineStore } from 'pinia'

export const useNewStore = defineStore('new', {
  state: () => ({
    // 状態
  }),
  actions: {
    // アクション
  },
  getters: {
    // ゲッター
  }
})
```

### 4. 設定項目の追加

```javascript
// src/config/defaults.js
export const DEFAULT_SETTINGS = {
  // ...既存設定
  newSetting: defaultValue
}
```

```vue
<!-- src/components/settings/NewSettings.vue -->
<template>
  <div class="setting-item">
    <label>新しい設定</label>
    <input v-model="settings.newSetting" />
  </div>
</template>

<script setup>
import { useSettingsStore } from '@/stores/settings'
const settings = useSettingsStore()
</script>
```

### Common Modification Points

- **新しい API プロバイダー**: `services/api/` にファイル追加、`config/providers.js` に定義追加
- **UI 変更**: `components/` のコンポーネント編集、`assets/styles/` の CSS 変更
- **新機能**: 適切なディレクトリに追加（`services/features/` 等）
- **設定**: `config/defaults.js` に追加、UI コンポーネント作成、Store で管理

---

## Deployment

### GitHub Pages (Automatic Deployment)

このプロジェクトは GitHub Actions で自動デプロイされます。

1. **main ブランチにプッシュ**すると自動的にビルド＆デプロイ
2. **公開 URL**: `https://<username>.github.io/<repo-name>/`
3. **Service Worker**: 自動的にキャッシュを管理、オフライン対応

### Manual Deployment

```bash
# ビルド
pnpm build

# dist/ ディレクトリを静的ホスティングサービスにデプロイ
# GitHub Pages, Netlify, Vercel 等
```

### Cache Management

Service Worker のキャッシュにより、ユーザーはすぐに更新を確認できない場合があります。

更新を強制する方法：
- GitHub Actions で自動デプロイされた際、新しいバージョンが自動的に配信されます
- ユーザーは設定 → キャッシュ管理から手動更新可能
- エラー閾値（1 分に 3 回エラー）で自動更新がトリガーされます

---

## Smartphone Support

**このアプリケーションはスマートフォンでの利用をメインに設計されています。**

### PWA Features

✅ **ホーム画面に追加可能**: PWA としてインストール可能
✅ **オフライン動作**: Service Worker でキャッシュ
✅ **レスポンシブデザイン**: 全画面サイズ対応
✅ **タッチ操作最適化**: スワイプナビゲーション、Passive Event Listener
✅ **パフォーマンス**: 60fps 維持、0.8 秒以下の初回表示

### Testing on Mobile

GitHub Pages にデプロイ後、スマートフォンのブラウザから直接アクセス可能：
- iOS Safari
- Android Chrome
- その他主要ブラウザ

---

## Configuration Files

### 設定ファイルの場所

すべての設定は `src/config/` ディレクトリに集約されています：

- **`defaults.js`**: デフォルト設定値（UI、API パラメータ等）
- **`themes.js`**: テーマ定義（カラースキーム、CSS 変数）
- **`providers.js`**: API プロバイダー定義（名前、デフォルトモデル、色）
- **`models.js`**: 各プロバイダーのモデルリスト
- **`constants.js`**: 定数定義

これらのファイルを編集することで、アプリケーション全体の挙動を変更できます。

---

## Coding Guidelines

### Vue.js Best Practices

- **Composition API** を使用
- **`<script setup>`** 構文を優先
- **TypeScript**: 段階的導入可能（オプション）

### Naming Conventions

- **コンポーネント**: PascalCase (`ChatMessage.vue`)
- **関数**: camelCase (`loadSession`)
- **定数**: UPPER_SNAKE_CASE (`MAX_MESSAGES`)
- **ファイル名**: コンポーネントは PascalCase、その他は kebab-case

### State Management Rules

- **グローバル状態**: すべて Pinia Store で管理
- **ローカル状態**: コンポーネント内の `ref` は局所的な状態のみ
- **Store 分割**: 機能ごとに Store を分離
- **循環参照禁止**: Store 間の循環依存を避ける

---

## License

MIT License

---

## Additional Resources

詳細な機能要件については、以下のドキュメントを参照してください：

- **`機能要件定義書_Vue版.md`**: 完全な機能要件とアーキテクチャ設計
- **`README.md`**: ユーザー向けドキュメント

---

**重要**: このプロジェクトで作業する際は、**必ず日本語で応答してください**。コメント、コミットメッセージ、ドキュメントもすべて日本語で記述します。
