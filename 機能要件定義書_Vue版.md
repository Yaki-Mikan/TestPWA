# GeminiPWA 機能要件定義書（Vue.js リアーキテクチャ版）

## 📌 ドキュメント情報

- **バージョン**: 2.0
- **対象**: 新規プロジェクトでの再実装
- **ベース**: GeminiPWA（Vanilla JS版）の機能を踏襲
- **主な変更**: Vue.js 3ベース、パフォーマンス最適化、モジュール化

---

## 目次

1. [プロジェクト概要](#1-プロジェクト概要)
2. [技術スタック](#2-技術スタック)
3. [アーキテクチャ設計](#3-アーキテクチャ設計)
4. [パフォーマンス要件](#4-パフォーマンス要件)
5. [機能要件](#5-機能要件)
6. [非機能要件](#6-非機能要件)
7. [デプロイメント](#7-デプロイメント)
8. [開発ガイドライン](#8-開発ガイドライン)

---

## 1. プロジェクト概要

### 1.1 プロジェクト目的

**既存のGeminiPWA（Vanilla JS版）を、Vue.js 3ベースで再構築する。**

- メンテナンス性の向上（単一ファイル→モジュール化）
- パフォーマンス最適化（特にスマートフォンでの利用）
- 機能の踏襲（既存機能をすべて引き継ぐ）
- 拡張性の確保（新機能追加が容易な設計）

### 1.2 アプリケーション基本情報

- **アプリケーション名**: GeminiPWA (Vue Edition)
- **種類**: Progressive Web Application (PWA)
- **アーキテクチャ**: Single Page Application (SPA)
- **主要用途**: 複数AIプロバイダーとのチャットインターフェース
- **ターゲットデバイス**: **スマートフォン（メイン）** + デスクトップ

### 1.3 開発方針

| 項目 | 方針 |
|------|------|
| **フレームワーク** | Vue.js 3（Composition API） |
| **状態管理** | Pinia（Vue公式） |
| **ビルドツール** | Vite |
| **モジュール化** | 機能ごとにパッケージング |
| **設定管理** | 別ファイルで管理（`config/`） |
| **パフォーマンス** | スマホ最適化を最優先 |

---

## 2. 技術スタック

### 2.1 コアフレームワーク・ライブラリ

| カテゴリ | 技術 | バージョン | バンドルサイズ | 選定理由 |
|---------|------|-----------|--------------|---------|
| **フレームワーク** | Vue.js | 3.4+ | ~34KB (gzip) | リアクティブ、軽量、成熟 |
| **状態管理** | Pinia | 2.1+ | ~8KB | Vue 3公式推奨 |
| **ルーティング** | Vue Router | 4.2+ | ~16KB | SPA標準 |
| **ビルドツール** | Vite | 5.0+ | - | 高速、ESM対応 |

### 2.2 UI/UXライブラリ

| カテゴリ | 技術 | バージョン | バンドルサイズ | 選定理由 |
|---------|------|-----------|--------------|---------|
| **仮想スクロール** | vue-virtual-scroller | 2.0+ | ~8KB | 大量メッセージ対応 |
| **Markdown** | markdown-it | 14.0+ | ~20KB | 高速、拡張可能（marked.js代替） |
| **サニタイザ** | DOMPurify | 3.0+ | ~45KB | XSS防止（継続使用） |
| **日付処理** | day.js | 1.11+ | ~2KB | 軽量（moment.js代替） |

### 2.3 データ管理

| カテゴリ | 技術 | バージョン | バンドルサイズ | 選定理由 |
|---------|------|-----------|--------------|---------|
| **IndexedDB** | idb | 8.0+ | ~3KB | Promise対応wrapper |

### 2.4 PWA関連

| カテゴリ | 技術 | 選定理由 |
|---------|------|---------|
| **Service Worker** | vite-plugin-pwa | 自動生成、Workbox統合 |
| **Manifest** | 手動作成 | PWA標準 |

### 2.5 開発ツール

| カテゴリ | 技術 | 用途 |
|---------|------|------|
| **パッケージ管理** | pnpm | 高速、ディスク効率的 |
| **リンター** | ESLint | コード品質維持 |
| **フォーマッター** | Prettier | コード整形 |
| **型チェック** | TypeScript (optional) | 型安全性（段階的導入可） |

### 2.6 総バンドルサイズ見積もり

| カテゴリ | サイズ（gzip圧縮後） |
|---------|-------------------|
| Vue.js関連 | ~58KB |
| UI/UXライブラリ | ~75KB |
| データ管理 | ~3KB |
| **合計（初期ロード）** | **~80KB** |
| **遅延ロード可能** | ~50KB |

**最終目標**: 初回ロード **80KB以下**（現行比 60%削減）

---

## 3. アーキテクチャ設計

### 3.1 プロジェクト構成

```
geminipwa-vue/
├── .github/
│   └── workflows/
│       └── deploy.yml          # GitHub Actions自動デプロイ
├── public/
│   ├── manifest.json           # PWA manifest
│   ├── robots.txt
│   └── icons/                  # アイコン各サイズ
│       ├── icon-72x72.png
│       ├── icon-96x96.png
│       ├── icon-128x128.png
│       ├── icon-144x144.png
│       ├── icon-152x152.png
│       ├── icon-192x192.png
│       ├── icon-384x384.png
│       └── icon-512x512.png
├── src/
│   ├── main.js                 # エントリーポイント
│   ├── App.vue                 # ルートコンポーネント
│   │
│   ├── router/                 # Vue Router設定
│   │   └── index.js
│   │
│   ├── stores/                 # Pinia状態管理
│   │   ├── chat.js             # チャット状態
│   │   ├── session.js          # セッション管理
│   │   ├── settings.js         # 設定管理
│   │   ├── apiProvider.js      # APIプロバイダー状態
│   │   ├── twinEngine.js       # Twin-Engine状態
│   │   └── ui.js               # UI状態（モーダル、サイドバー等）
│   │
│   ├── views/                  # ページコンポーネント
│   │   ├── ChatView.vue        # チャット画面
│   │   ├── HistoryView.vue     # 履歴一覧画面
│   │   └── SettingsView.vue    # 設定画面
│   │
│   ├── components/             # 再利用可能コンポーネント
│   │   ├── chat/
│   │   │   ├── ChatContainer.vue       # チャット全体コンテナ
│   │   │   ├── ChatMessages.vue        # メッセージ一覧（仮想スクロール）
│   │   │   ├── ChatMessage.vue         # 単一メッセージ
│   │   │   ├── MessageBubble.vue       # 吹き出し
│   │   │   ├── MessageActions.vue      # メッセージ操作ボタン
│   │   │   ├── ChatInput.vue           # 入力欄
│   │   │   ├── AttachmentButton.vue    # 添付ボタン
│   │   │   ├── ThoughtProcess.vue      # 思考プロセス表示
│   │   │   └── StreamingText.vue       # ストリーミングテキスト
│   │   │
│   │   ├── session/
│   │   │   ├── SessionList.vue         # セッション一覧
│   │   │   ├── SessionItem.vue         # セッションアイテム
│   │   │   └── SessionActions.vue      # セッション操作
│   │   │
│   │   ├── settings/
│   │   │   ├── SettingsPanel.vue       # 設定パネル
│   │   │   ├── ApiSettings.vue         # API設定
│   │   │   ├── ProviderSettings.vue    # プロバイダー別設定
│   │   │   ├── UiSettings.vue          # UI設定
│   │   │   ├── ThemeSettings.vue       # テーマ設定
│   │   │   ├── TwinEngineSettings.vue  # Twin-Engine設定
│   │   │   └── ProofreadingSettings.vue # 校正設定
│   │   │
│   │   ├── common/
│   │   │   ├── AppHeader.vue           # ヘッダー
│   │   │   ├── AppFooter.vue           # フッター
│   │   │   ├── Modal.vue               # モーダル
│   │   │   ├── Dropdown.vue            # ドロップダウン
│   │   │   ├── Button.vue              # ボタン
│   │   │   ├── IconButton.vue          # アイコンボタン
│   │   │   └── LoadingSpinner.vue      # ローディング
│   │   │
│   │   └── features/
│   │       ├── TwinEngineSummary.vue   # Twin-Engine要約エリア
│   │       ├── ClipboardStack.vue      # クリップボードスタック
│   │       ├── MemoArea.vue            # メモエリア
│   │       └── DiceRoller.vue          # ダイス機能
│   │
│   ├── services/               # ビジネスロジック層
│   │   ├── api/                # API通信サービス
│   │   │   ├── base.js         # 基底クラス
│   │   │   ├── gemini.js       # Gemini API
│   │   │   ├── deepseek.js     # DeepSeek API
│   │   │   ├── claude.js       # Claude API
│   │   │   ├── openai.js       # OpenAI API
│   │   │   ├── xai.js          # xAI API
│   │   │   ├── llmaggregator.js # LLM Aggregator
│   │   │   └── dummy.js        # Dummy AI
│   │   │
│   │   ├── db/                 # データベースサービス
│   │   │   ├── indexeddb.js    # IndexedDB操作
│   │   │   ├── sessionStore.js # セッション保存
│   │   │   └── settingsStore.js # 設定保存
│   │   │
│   │   ├── features/           # 高度な機能
│   │   │   ├── twinEngine.js   # Twin-Engine処理
│   │   │   ├── proofreading.js # 校正処理
│   │   │   ├── sessionLink.js  # セッションリンク
│   │   │   └── webhook.js      # Webhook通知
│   │   │
│   │   └── export/             # エクスポート・インポート
│   │       ├── exportSession.js
│   │       └── importSession.js
│   │
│   ├── composables/            # Vue Composables（共通ロジック）
│   │   ├── useMarkdown.js      # Markdown処理
│   │   ├── useStreaming.js     # ストリーミング処理
│   │   ├── useVirtualScroll.js # 仮想スクロール
│   │   ├── useTheme.js         # テーマ管理
│   │   ├── useSwipe.js         # スワイプ操作
│   │   └── useClipboard.js     # クリップボード操作
│   │
│   ├── utils/                  # ユーティリティ関数
│   │   ├── markdown.js         # Markdown解析
│   │   ├── sanitize.js         # サニタイズ
│   │   ├── storage.js          # ストレージ操作
│   │   ├── export.js           # データエクスポート
│   │   ├── import.js           # データインポート
│   │   ├── date.js             # 日付フォーマット
│   │   └── validators.js       # バリデーション
│   │
│   ├── workers/                # Web Workers
│   │   └── markdown.worker.js  # Markdown解析ワーカー
│   │
│   ├── config/                 # 設定ファイル
│   │   ├── defaults.js         # デフォルト設定値
│   │   ├── themes.js           # テーマ定義
│   │   ├── providers.js        # APIプロバイダー定義
│   │   ├── models.js           # モデルリスト
│   │   └── constants.js        # 定数定義
│   │
│   └── assets/                 # 静的リソース
│       ├── styles/
│       │   ├── main.css        # グローバルスタイル
│       │   ├── variables.css   # CSS変数
│       │   ├── themes.css      # テーマスタイル
│       │   └── animations.css  # アニメーション
│       └── images/
│           └── placeholder.png
│
├── .env.example                # 環境変数サンプル
├── .gitignore
├── package.json                # 依存関係定義
├── pnpm-lock.yaml
├── vite.config.js              # Viteビルド設定
├── eslint.config.js            # ESLint設定
├── prettier.config.js          # Prettier設定
└── README.md
```

### 3.2 状態管理設計（Pinia Stores）

#### 3.2.1 `stores/chat.js` - チャット状態

```javascript
// 管理する状態
{
  currentMessages: [],      // 現在のメッセージ配列
  isSending: false,         // 送信中フラグ
  isSummarizing: false,     // 要約中フラグ
  isProofreading: false,    // 校正中フラグ
  abortController: null,    // API中断用
  streamingText: '',        // ストリーミング中テキスト
  messageCollapsedStates: Map, // メッセージ折りたたみ状態
  thoughtStates: Map        // 思考プロセス開閉状態
}
```

#### 3.2.2 `stores/session.js` - セッション管理

```javascript
{
  currentSessionId: null,   // 現在のセッションID
  sessions: [],             // 全セッション一覧
  linkedSessionIds: [],     // リンクされたセッションID
  sortOrder: 'updatedAt'    // ソート順
}
```

#### 3.2.3 `stores/settings.js` - 設定管理

```javascript
{
  theme: 'light',           // テーマ
  apiProvider: 'gemini',    // 使用プロバイダー
  apiKeys: {},              // APIキー（プロバイダーごと）
  systemPrompts: {},        // システムプロンプト
  parameters: {},           // APIパラメータ
  ui: {},                   // UI設定
  advanced: {}              // 高度な設定
}
```

#### 3.2.4 `stores/twinEngine.js` - Twin-Engine状態

```javascript
{
  enabled: false,           // Twin-Engine有効化
  autoMode: false,          // 自動モード
  summaryContent: '',       // 要約内容
  apiConfigs: [],           // Twin-Engine用API設定
  activeConfigId: null      // アクティブ設定ID
}
```

### 3.3 ルーティング設計

```javascript
// router/index.js
{
  path: '/',
  redirect: '/chat'
},
{
  path: '/chat',
  name: 'chat',
  component: ChatView,
  meta: { title: 'チャット' }
},
{
  path: '/chat/:sessionId',
  name: 'chat-session',
  component: ChatView,
  meta: { title: 'チャット' }
},
{
  path: '/history',
  name: 'history',
  component: HistoryView,
  meta: { title: '履歴' }
},
{
  path: '/settings',
  name: 'settings',
  component: SettingsView,
  meta: { title: '設定' }
}
```

---

## 4. パフォーマンス要件

### 4.1 目標指標

| 指標 | 目標値 | 測定方法 |
|------|-------|---------|
| **初回表示時間（FCP）** | 0.8秒以下 | Lighthouse |
| **インタラクティブ時間（TTI）** | 1.5秒以下 | Lighthouse |
| **初期バンドルサイズ** | 80KB以下（gzip） | Vite bundle analyzer |
| **メモリ使用量（1000件表示時）** | 30MB以下 | Chrome DevTools |
| **スクロールFPS** | 60fps維持 | Chrome DevTools Performance |
| **IndexedDB読み込み** | 50ms以下 | console.time() |
| **Lighthouse Score** | 95点以上 | Lighthouse |

### 4.2 最適化手法

#### 4.2.1 仮想スクロール（Virtual Scrolling）

**要件ID**: PERF-001

**概要**:
大量メッセージを表示する際、画面に表示されているメッセージのみをDOMにレンダリング。

**実装**:
```vue
<template>
  <RecycleScroller
    :items="messages"
    :item-size="80"
    key-field="id"
    v-slot="{ item }"
  >
    <ChatMessage :message="item" />
  </RecycleScroller>
</template>
```

**効果**:
- メモリ使用量 90%削減
- スクロール速度 10倍向上
- 1万件のメッセージでも60fps維持

---

#### 4.2.2 Markdown解析の最適化

**要件ID**: PERF-002

**概要**:
Markdown解析結果をキャッシュし、重複解析を防ぐ。Web Workerで重い処理をバックグラウンド実行。

**実装**:
```javascript
// composables/useMarkdown.js
import { ref, computed } from 'vue'

const mdCache = new Map()

export function useMarkdown(text) {
  return computed(() => {
    if (mdCache.has(text)) return mdCache.get(text)

    const result = md.render(text)
    mdCache.set(text, result)

    // キャッシュサイズ制限（1000件まで）
    if (mdCache.size > 1000) {
      const firstKey = mdCache.keys().next().value
      mdCache.delete(firstKey)
    }

    return result
  })
}
```

**Web Worker対応**:
```javascript
// workers/markdown.worker.js
import { markdown } from './markdown'

self.addEventListener('message', (e) => {
  const html = markdown.render(e.data.text)
  self.postMessage({ id: e.data.id, html })
})
```

**効果**:
- 再レンダリング時の処理時間 95%削減
- UIブロッキングなし

---

#### 4.2.3 画像の遅延読み込み

**要件ID**: PERF-003

**概要**:
画像を画面に表示される直前に読み込む。

**実装**:
```vue
<template>
  <img
    v-lazy="imageUrl"
    :src="placeholder"
    loading="lazy"
    decoding="async"
  />
</template>
```

**効果**:
- 初期表示速度 50%向上
- データ通信量削減

---

#### 4.2.4 IndexedDBの最適化

**要件ID**: PERF-004

**概要**:
ページネーション、インデックス最適化、メッセージの別ストア化。

**実装**:
```javascript
// services/db/indexeddb.js
import { openDB } from 'idb'

const db = await openDB('GeminiPWADB', 2, {
  upgrade(db, oldVersion, newVersion, transaction) {
    // セッションストア
    if (!db.objectStoreNames.contains('sessions')) {
      const sessionStore = db.createObjectStore('sessions', {
        keyPath: 'id',
        autoIncrement: true
      })
      sessionStore.createIndex('updatedAt', 'updatedAt')
      sessionStore.createIndex('createdAt', 'createdAt')
    }

    // メッセージストア（分離）
    if (!db.objectStoreNames.contains('messages')) {
      const messageStore = db.createObjectStore('messages', {
        keyPath: 'id',
        autoIncrement: true
      })
      messageStore.createIndex('sessionId', 'sessionId')
      messageStore.createIndex('timestamp', 'timestamp')
    }

    // 設定ストア
    if (!db.objectStoreNames.contains('settings')) {
      db.createObjectStore('settings', { keyPath: 'key' })
    }
  }
})

// ページネーション対応読み込み
export async function loadSessions(limit = 20, offset = 0) {
  const sessions = await db.getAllFromIndex(
    'sessions',
    'updatedAt',
    null,
    limit
  )
  return sessions.reverse()
}
```

**効果**:
- 読み込み速度 5倍向上
- 無限スクロール対応

---

#### 4.2.5 コード分割（Code Splitting）

**要件ID**: PERF-005

**概要**:
使用する機能のみを読み込み、未使用のコードを遅延ロード。

**実装**:
```javascript
// vite.config.js
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'vendor-vue': ['vue', 'vue-router', 'pinia'],
          'vendor-ui': ['markdown-it', 'dompurify', 'vue-virtual-scroller'],
          'api-gemini': ['./src/services/api/gemini.js'],
          'api-deepseek': ['./src/services/api/deepseek.js'],
          'api-claude': ['./src/services/api/claude.js'],
          'api-openai': ['./src/services/api/openai.js'],
          'api-xai': ['./src/services/api/xai.js'],
          'api-llmaggregator': ['./src/services/api/llmaggregator.js'],
        }
      }
    }
  }
})
```

**動的インポート**:
```javascript
// services/api/base.js
export async function loadProvider(providerName) {
  const providers = {
    gemini: () => import('./gemini.js'),
    deepseek: () => import('./deepseek.js'),
    claude: () => import('./claude.js'),
    openai: () => import('./openai.js'),
    xai: () => import('./xai.js'),
    llmaggregator: () => import('./llmaggregator.js'),
  }

  const module = await providers[providerName]()
  return module.default
}
```

**効果**:
- 初期読み込みサイズ 60%削減
- 使用プロバイダーのみ読み込み

---

#### 4.2.6 リアクティブの最適化

**要件ID**: PERF-006

**概要**:
不要な深い監視を避け、shallowRefやshallowReactiveを使用。

**実装**:
```vue
<script setup>
import { shallowRef, shallowReactive, computed } from 'vue'

// 深い監視が不要な大量データ
const messages = shallowRef([])

// 設定オブジェクト（ネストが浅い）
const settings = shallowReactive({
  theme: 'light',
  provider: 'gemini'
})

// 表示用データのみ計算
const visibleMessages = computed(() => {
  return messages.value.slice(-50) // 最新50件のみ
})
</script>
```

**効果**:
- リアクティブ更新のオーバーヘッド削減
- メモリ使用量 20%削減

---

#### 4.2.7 デバウンス・スロットリング

**要件ID**: PERF-007

**概要**:
頻繁に発火するイベント（スクロール、リサイズ、入力）を制御。

**実装**:
```javascript
// composables/useDebounce.js
import { ref, watch } from 'vue'

export function useDebouncedRef(value, delay = 300) {
  const debounced = ref(value)
  let timeout

  watch(() => value, (newValue) => {
    clearTimeout(timeout)
    timeout = setTimeout(() => {
      debounced.value = newValue
    }, delay)
  })

  return debounced
}
```

```vue
<template>
  <input v-model="searchQuery" />
</template>

<script setup>
import { ref } from 'vue'
import { useDebouncedRef } from '@/composables/useDebounce'

const searchQuery = ref('')
const debouncedQuery = useDebouncedRef(searchQuery, 300)

// debouncedQueryを使って検索処理
</script>
```

---

#### 4.2.8 Service Workerキャッシュ戦略

**要件ID**: PERF-008

**概要**:
静的リソースとAPIレスポンスで異なるキャッシュ戦略を適用。

**実装**:
```javascript
// vite.config.js
VitePWA({
  workbox: {
    runtimeCaching: [
      {
        // 静的リソース: キャッシュ優先
        urlPattern: /\.(?:js|css|woff2|png|jpg|svg)$/,
        handler: 'CacheFirst',
        options: {
          cacheName: 'static-resources',
          expiration: {
            maxEntries: 100,
            maxAgeSeconds: 60 * 60 * 24 * 30 // 30日
          }
        }
      },
      {
        // API通信: ネットワーク優先
        urlPattern: /^https:\/\/.*\.googleapis\.com/,
        handler: 'NetworkFirst',
        options: {
          cacheName: 'api-cache',
          expiration: {
            maxEntries: 50,
            maxAgeSeconds: 60 * 5 // 5分
          },
          networkTimeoutSeconds: 10
        }
      }
    ]
  }
})
```

**効果**:
- オフライン対応
- 2回目以降の表示速度 80%向上

---

#### 4.2.9 プリフェッチ・プリロード

**要件ID**: PERF-009

**概要**:
次に必要になりそうなリソースを事前読み込み。

**実装**:
```javascript
// router/index.js
router.beforeEach(async (to, from, next) => {
  // 次のセッションをプリフェッチ
  if (to.name === 'chat' && to.params.sessionId) {
    prefetchSession(to.params.sessionId)
  }

  // 設定画面に遷移する際、設定を事前読み込み
  if (to.name === 'settings') {
    prefetchSettings()
  }

  next()
})

async function prefetchSession(sessionId) {
  // バックグラウンドで読み込み
  const session = await db.sessions.get(sessionId)
  // キャッシュに保存
}
```

---

#### 4.2.10 Brotli圧縮

**要件ID**: PERF-010

**概要**:
Gzipより高圧縮率のBrotli圧縮を使用。

**実装**:
```javascript
// vite.config.js
import compression from 'vite-plugin-compression'

export default defineConfig({
  plugins: [
    compression({
      algorithm: 'brotliCompress',
      ext: '.br',
      threshold: 1024 // 1KB以上のファイルを圧縮
    })
  ]
})
```

**効果**:
- バンドルサイズ 15-20%追加削減

---

### 4.3 スマートフォン特化最適化

#### 4.3.1 タッチ操作最適化

**要件ID**: PERF-011

**概要**:
Passive Event Listenerでスクロール性能向上。

**実装**:
```vue
<template>
  <div
    @touchstart.passive="onTouchStart"
    @touchmove.passive="onTouchMove"
    @touchend.passive="onTouchEnd"
  >
    <!-- コンテンツ -->
  </div>
</template>
```

**効果**:
- スクロール時のブロッキング解消
- 60fps維持

---

#### 4.3.2 レスポンシブ画像

**要件ID**: PERF-012

**概要**:
デバイスに応じた画像サイズを提供。

**実装**:
```vue
<template>
  <img
    :srcset="`${image}-sm.jpg 480w, ${image}-md.jpg 768w, ${image}-lg.jpg 1024w`"
    sizes="(max-width: 480px) 480px, (max-width: 768px) 768px, 1024px"
    :src="`${image}-md.jpg`"
  />
</template>
```

---

#### 4.3.3 ネットワーク状態に応じた最適化

**要件ID**: PERF-013

**概要**:
低速回線時の自動調整。

**実装**:
```javascript
// composables/useNetworkStatus.js
import { ref, onMounted } from 'vue'

export function useNetworkStatus() {
  const isSlowConnection = ref(false)

  onMounted(() => {
    const connection = navigator.connection
    if (connection) {
      isSlowConnection.value = connection.effectiveType === '2g' ||
                                connection.effectiveType === 'slow-2g'

      connection.addEventListener('change', () => {
        isSlowConnection.value = connection.effectiveType === '2g' ||
                                  connection.effectiveType === 'slow-2g'
      })
    }
  })

  return { isSlowConnection }
}
```

```vue
<script setup>
import { useNetworkStatus } from '@/composables/useNetworkStatus'

const { isSlowConnection } = useNetworkStatus()

// 低速時はストリーミング無効化、画像品質低下等
</script>
```

---

### 4.4 パフォーマンス監視

**要件ID**: PERF-014

**概要**:
開発時・本番環境でのパフォーマンス監視。

**実装**:
```javascript
// utils/performance.js
export function measurePerformance(label, fn) {
  const start = performance.now()
  const result = fn()
  const end = performance.now()

  console.log(`[Performance] ${label}: ${(end - start).toFixed(2)}ms`)

  return result
}

// 使用例
measurePerformance('Load Session', () => {
  return db.sessions.get(sessionId)
})
```

---

## 5. 機能要件

### 5.1 コア機能

#### 5.1.1 基本チャット機能

**要件ID**: FUNC-001

**継承元**: 既存GeminiPWAの機能を完全踏襲

**詳細**:

1. **メッセージ送信**
   - テキスト入力
   - Enterキー送信（設定可能）
   - ファイル・画像添付
   - 状態管理: Pinia `chat` store

2. **メッセージ表示**
   - Markdown対応（markdown-it）
   - サニタイズ（DOMPurify）
   - コードブロック（シンタックスハイライト）
   - Mermaid図
   - 仮想スクロール対応

3. **ストリーミング表示**
   - Server-Sent Events (SSE)
   - リアルタイム文字表示
   - 速度調整可能

**コンポーネント**:
- `ChatView.vue`
- `ChatContainer.vue`
- `ChatMessages.vue`
- `ChatMessage.vue`
- `ChatInput.vue`

---

#### 5.1.2 セッション管理

**要件ID**: FUNC-002

**詳細**:

1. **マルチセッション**
   - 複数セッションの独立管理
   - セッション切替（スワイプ対応）
   - ソート順変更（更新日時/作成日時）

2. **セッション操作**
   - 新規作成
   - 複製
   - 削除
   - エクスポート/インポート（JSON）
   - 一括操作

**コンポーネント**:
- `HistoryView.vue`
- `SessionList.vue`
- `SessionItem.vue`

**Store**:
- `stores/session.js`

---

#### 5.1.3 メッセージ操作

**要件ID**: FUNC-003

**詳細**:

1. **編集**: ユーザー・AIメッセージの編集
2. **削除**: 単一/カスケード削除
3. **再試行**: AI応答の再生成
4. **複製**: メッセージ複製
5. **コピー**: テキスト/コードブロックコピー
6. **折りたたみ**: 上部/下部ボタン、状態保持

**コンポーネント**:
- `MessageActions.vue`

---

### 5.2 API連携

**要件ID**: FUNC-004

**対応プロバイダー**:

1. **Google Gemini**
   - ファイル解析、Grounding、思考プロセス
2. **DeepSeek**
   - 思考プロセス、カスタムエンドポイント
3. **Anthropic Claude**
   - Extended Thinking
4. **OpenAI**
   - GPT-4o等
5. **xAI (Grok)**
   - Vision、推論努力レベル
6. **LLM Aggregator**
   - OpenRouter等
7. **Dummy AI**
   - テスト・デバッグ用

**実装**:
- 各プロバイダーを`services/api/`に分離
- 基底クラス`base.js`で共通処理
- 動的インポートでコード分割

**コンポーネント**:
- `ProviderSettings.vue`

**Store**:
- `stores/apiProvider.js`

---

### 5.3 データ管理

**要件ID**: FUNC-005

**詳細**:

1. **IndexedDB**
   - セッション保存（`sessions` store）
   - メッセージ保存（`messages` store）
   - 設定保存（`settings` store）
   - ページネーション対応

2. **エクスポート/インポート**
   - JSON形式
   - 単一/一括操作

3. **添付ファイル**
   - Base64保存
   - プレビュー表示

**サービス**:
- `services/db/indexeddb.js`
- `services/export/`

---

### 5.4 UI/UX機能

**要件ID**: FUNC-006

**詳細**:

1. **テーマ**
   - 7種類（light, dark, pastel系5種）
   - カスタム背景画像

2. **カスタマイズ**
   - フォント設定
   - 透明度調整
   - レイアウト調整
   - アイコン・名前表示

3. **ボタン表示制御**
   - ヘッダー/フッターボタンの表示/非表示

**コンポーネント**:
- `SettingsView.vue`
- `ThemeSettings.vue`
- `UiSettings.vue`

**設定ファイル**:
- `config/themes.js`
- `config/defaults.js`

---

### 5.5 高度な機能

#### 5.5.1 Twin-Engine（自動要約）

**要件ID**: FUNC-007

**詳細**:
- 2つのAPI使い分け（会話用・要約用）
- 自動/手動モード
- 要約プロンプトカスタマイズ
- トークン削減

**サービス**:
- `services/features/twinEngine.js`

**Store**:
- `stores/twinEngine.js`

**コンポーネント**:
- `TwinEngineSummary.vue`
- `TwinEngineSettings.vue`

---

#### 5.5.2 セッションリンク（AI間会話モード）

**要件ID**: FUNC-008

**詳細**:
- 2セッションをリンク
- AI→AIメッセージ転送
- リンク状態の可視化

**サービス**:
- `services/features/sessionLink.js`

---

#### 5.5.3 校正機能

**要件ID**: FUNC-009

**詳細**:
- 応答生成後に別モデルで校正
- 校正用API設定管理

**サービス**:
- `services/features/proofreading.js`

**コンポーネント**:
- `ProofreadingSettings.vue`

---

#### 5.5.4 思考プロセス表示

**要件ID**: FUNC-010

**詳細**:
- DeepSeek, Gemini, Claude, xAI対応
- 折りたたみ/展開
- 専用スタイル

**コンポーネント**:
- `ThoughtProcess.vue`

---

#### 5.5.5 Geminiプリセット

**要件ID**: FUNC-011

**詳細**:
- 設定プリセット保存・切替
- Gemini専用

---

#### 5.5.6 複数APIキー管理

**要件ID**: FUNC-012

**詳細**:
- プロバイダーごとに複数キー登録
- 切替機能

---

### 5.6 その他の機能

**要件ID**: FUNC-013

**詳細**:

1. **メモ機能**: 一時メモ保存
2. **クリップボードスタック**: コピー内容蓄積
3. **ダイス機能**: ランダム数値生成
4. **Webhook通知**: 応答完了時の外部通知

**コンポーネント**:
- `MemoArea.vue`
- `ClipboardStack.vue`
- `DiceRoller.vue`

---

## 6. 非機能要件

### 6.1 セキュリティ

**要件ID**: NFR-001

**詳細**:

1. **APIキー保護**
   - IndexedDBにローカル保存のみ
   - サーバー送信なし
   - 環境変数での管理オプション

2. **XSS防止**
   - DOMPurifyでサニタイズ
   - CSP設定

3. **HTTPS強制**
   - GitHub Pagesは自動HTTPS

---

### 6.2 パフォーマンス

**要件ID**: NFR-002

詳細は [4. パフォーマンス要件](#4-パフォーマンス要件) を参照。

---

### 6.3 互換性

**要件ID**: NFR-003

**対応ブラウザ**:

| ブラウザ | 最低バージョン |
|---------|--------------|
| Chrome | 90+ |
| Firefox | 88+ |
| Safari | 14+ |
| Edge | 90+ |

**必須機能**:
- IndexedDB
- Service Worker
- ES2015+

---

### 6.4 アクセシビリティ

**要件ID**: NFR-004

**詳細**:

1. **キーボード操作**: 全機能をキーボードで操作可能
2. **スクリーンリーダー**: ARIA属性の適切な使用
3. **コントラスト**: WCAG AA準拠

---

### 6.5 データ移行

**要件ID**: NFR-005

**詳細**:

既存GeminiPWA（Vanilla JS版）からのデータ移行ツール提供。

**実装**:
```javascript
// utils/migration.js
export async function migrateFromVanillaVersion() {
  // 既存IndexedDBからデータ読み込み
  const oldDb = await openDB('GeminiPWADB', 1)
  const oldSessions = await oldDb.getAll('chats')
  const oldSettings = await oldDb.getAll('settings')

  // 新形式に変換
  const newSessions = oldSessions.map(convertSession)
  const newSettings = convertSettings(oldSettings)

  // 新DBに保存
  // ...
}
```

---

## 7. デプロイメント

### 7.1 開発環境

**要件**:

1. **ローカル開発サーバー**
   ```bash
   pnpm install
   pnpm dev
   ```

2. **ホットリロード**: Viteの自動リロード

3. **デバッグ**: Vue DevToolsで状態確認

---

### 7.2 ビルド

**要件**:

```bash
pnpm build
```

**出力**:
- `dist/` ディレクトリに静的ファイル生成
- `dist/index.html`
- `dist/assets/` (JS, CSS)
- `dist/manifest.json`
- `dist/sw.js` (Service Worker)

---

### 7.3 本番環境（GitHub Pages）

**要件**:

1. **自動デプロイ**: GitHub Actionsで自動化

```yaml
# .github/workflows/deploy.yml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - uses: pnpm/action-setup@v2
        with:
          version: 8

      - uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Build
        run: pnpm build

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
```

2. **公開URL**: `https://<username>.github.io/<repo-name>/`

3. **カスタムドメイン**: オプション

---

### 7.4 スマートフォン対応

**要件**:

1. ✅ **PWAインストール**: ホーム画面に追加可能
2. ✅ **オフライン動作**: Service Workerでキャッシュ
3. ✅ **レスポンシブデザイン**: 全画面サイズ対応
4. ✅ **タッチ操作**: スワイプナビゲーション等

**確認項目**:
- [ ] Lighthouse PWAスコア 90点以上
- [ ] iOS Safariで動作確認
- [ ] Android Chromeで動作確認
- [ ] ホーム画面追加テスト

---

## 8. 開発ガイドライン

### 8.1 コーディング規約

**要件**:

1. **Vue.js**
   - Composition API使用
   - `<script setup>`構文
   - TypeScript導入（段階的、optional）

2. **命名規則**
   - コンポーネント: PascalCase（`ChatMessage.vue`）
   - 関数: camelCase（`loadSession`）
   - 定数: UPPER_SNAKE_CASE（`MAX_MESSAGES`）

3. **コメント**
   - 複雑なロジックにはコメント必須
   - JSDocで関数説明

---

### 8.2 状態管理規約

**要件**:

1. **Pinia Store使用**
   - グローバル状態はすべてStoreで管理
   - コンポーネント内の`ref`は局所的な状態のみ

2. **Store分割**
   - 機能ごとにStore分離
   - 循環参照禁止

---

### 8.3 テスト（推奨）

**要件**:

1. **単体テスト**: Vitest
2. **E2Eテスト**: Playwright（オプション）

```bash
pnpm test:unit
pnpm test:e2e
```

---

### 8.4 Git運用

**要件**:

1. **ブランチ戦略**
   - `main`: 本番
   - `develop`: 開発
   - `feature/*`: 機能開発

2. **コミットメッセージ**
   ```
   feat: 新機能追加
   fix: バグ修正
   perf: パフォーマンス改善
   refactor: リファクタリング
   docs: ドキュメント
   style: コードスタイル
   test: テスト
   ```

---

## 9. パッケージ定義

### 9.1 package.json

```json
{
  "name": "geminipwa-vue",
  "version": "2.0.0",
  "description": "Multi-AI Chat PWA built with Vue.js 3",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext .vue,.js,.jsx,.cjs,.mjs --fix",
    "format": "prettier --write src/",
    "test:unit": "vitest",
    "type-check": "vue-tsc --noEmit"
  },
  "dependencies": {
    "vue": "^3.4.21",
    "vue-router": "^4.3.0",
    "pinia": "^2.1.7",
    "vue-virtual-scroller": "^2.0.0-beta.8",
    "markdown-it": "^14.1.0",
    "dompurify": "^3.0.11",
    "dayjs": "^1.11.10",
    "idb": "^8.0.0"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^5.0.4",
    "vite": "^5.2.0",
    "vite-plugin-pwa": "^0.19.8",
    "vite-plugin-compression": "^0.5.1",
    "eslint": "^8.57.0",
    "eslint-plugin-vue": "^9.24.0",
    "prettier": "^3.2.5",
    "vitest": "^1.4.0",
    "@vue/test-utils": "^2.4.5"
  },
  "engines": {
    "node": ">=18.0.0",
    "pnpm": ">=8.0.0"
  }
}
```

---

## 10. マイルストーン

### Phase 1: 基盤構築（1-2週間）

- [x] プロジェクトセットアップ
- [x] ルーティング設定
- [x] Pinia Store構築
- [x] IndexedDB移行
- [x] 基本UI構築

### Phase 2: コア機能実装（2-3週間）

- [ ] チャット機能
- [ ] セッション管理
- [ ] API連携（7プロバイダー）
- [ ] Markdown表示
- [ ] 仮想スクロール

### Phase 3: 高度な機能（2週間）

- [ ] Twin-Engine
- [ ] セッションリンク
- [ ] 校正機能
- [ ] 思考プロセス表示
- [ ] Geminiプリセット

### Phase 4: 最適化・PWA（1週間）

- [ ] パフォーマンス最適化
- [ ] コード分割
- [ ] Service Worker
- [ ] PWA manifest

### Phase 5: テスト・デプロイ（1週間）

- [ ] 単体テスト
- [ ] スマホ実機テスト
- [ ] Lighthouse監査
- [ ] GitHub Pages デプロイ

---

## 11. 既存版との比較

| 項目 | Vanilla JS版 | Vue.js版 |
|------|-------------|---------|
| **ファイル数** | 1ファイル | 100+ファイル |
| **総行数** | ~17,000行 | ~10,000行（見込み） |
| **バンドルサイズ** | ~200KB | ~80KB |
| **初回表示** | 2.5秒 | 0.8秒 |
| **メモリ（1000件）** | 150MB | 30MB |
| **スクロールFPS** | 30fps | 60fps |
| **メンテナンス性** | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| **拡張性** | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| **パフォーマンス** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

---

## 12. リスク・制約事項

### 12.1 技術的リスク

| リスク | 影響度 | 対策 |
|-------|-------|------|
| IndexedDBマイグレーション失敗 | 高 | データ移行ツールの十分なテスト |
| パフォーマンス目標未達 | 中 | 段階的最適化、代替案準備 |
| ブラウザ互換性問題 | 中 | Polyfill使用、対象ブラウザ限定 |

### 12.2 制約事項

1. **ビルドプロセス必須**: ビルド前のソースコードは直接動作しない
2. **Node.js環境必要**: 開発時にNode.js 18+が必須
3. **データ移行**: 既存版からの手動移行が必要

---

## 改訂履歴

| バージョン | 日付 | 変更内容 | 作成者 |
|----------|------|---------|-------|
| 2.0 | 2025年 | Vue.js版として新規作成 | - |

---

**以上**
