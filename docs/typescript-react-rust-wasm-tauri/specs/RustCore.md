# Rust Core

## 目次

- [概要](#概要)
- [想定するモジュール構成](#想定するモジュール構成)
- [Rust Core に入れるもの](#rust-core-に入れるもの)
  - [ノード実行エンジン](#ノード実行エンジン)
  - [ワークフロー実行](#ワークフロー実行)
  - [JSON Schema](#json-schema)
  - [ポーズ補正](#ポーズ補正)
- [Rust Core に入れないもの](#rust-core-に入れないもの)
  - [React UI](#react-ui)
  - [Tauri 固有処理](#tauri-固有処理)
  - [Web API](#web-api)
- [なぜ分けるのか](#なぜ分けるのか)
  - [Web版](#web版)
  - [Tauri版](#tauri版)
  - [Server版](#server版)
- [推奨する構成](#推奨する構成)

---

<style>
a {
    text-decoration: none;
    color: #90CAF9;
}
a:hover {
    text-decoration: underline;
    color: #FFFFFF;
}
</style>

## TODO リスト

Rust Core 実装の進捗：

- [x] アーキテクチャ設計
- [x] ノード実行エンジン
- [x] ワークフロー実行システム
- [x] JSON Schema 定義
- [x] ポーズ補正処理
- [ ] エラーハンドリング実装
- [ ] パフォーマンス最適化
- [ ] ユニットテスト
- [ ] インテグレーションテスト
- [ ] ドキュメント例の充実
- [ ] デバッグツール開発

---

## 概要

**Rust Core** は特定の技術製品名ではなく、アプリケーションの**中核ロジックを Rust で実装した共通ライブラリ**を指す設計パターンです。

ノードベースの AI 制作ツールを例にすると、次の構造になります。

```text
UI (React / TypeScript)
        ↓
Rust Core
        ↓
各種エンジン
```

## 想定するモジュール構成

```text
rust-core/
├─ node-engine
├─ workflow-engine
├─ pose-engine
├─ sprite-engine
├─ project-model
└─ json-schema
```

## Rust Core に入れるもの

### ノード実行エンジン

ノードの実行とグラフ全体の処理を担当します。

- ノードを実行する
- グラフを実行する
- ノード間の接続を検証する

Rust では、たとえば次の処理に相当します。

```rust
ExecuteNode
ExecuteGraph
ValidateConnection
```

### ワークフロー実行

ワークフロー制御に必要な処理を担当します。

- ループ処理
- 条件分岐
- 順次実行

```rust
LoopNode
IfNode
SequenceNode
```

### JSON Schema

アプリケーションが扱う主要なデータ構造を定義します。

- `Workflow`
- `Node`
- `Port`
- `Connection`

これらを共有することで、シリアライズ、検証、型の整合性を一箇所で管理できます。

### ポーズ補正

キャラクターやスプライトの補正処理も Rust Core に含めます。

- 頭サイズ補正
- 手足サイズ補正
- スプライト正規化

## Rust Core に入れないもの

### React UI

画面描画やユーザー操作は Rust Core に含めません。

- React
- Mantine
- MUI
- Tailwind

### Tauri 固有処理

Tauri API の呼び出しやデスクトップ固有機能は、Rust Core と分離します。

```rust
#[tauri::command]
```

### Web API

Axum や Actix などの Web サーバーフレームワーク固有の処理は、Rust Core に含めません。

- Axum
- Actix

## なぜ分けるのか

ポーズ補正処理を一度実装すると、複数の配布形態で同じ処理を利用できます。

### Web版

```text
React
  ↓
WASM
  ↓
Rust Core
```

### Tauri版

```text
React
  ↓
Tauri
  ↓
Rust Core
```

### Server版

```text
API
  ↓
Rust Core
```

これにより、実装の重複や処理差異を減らせます。

## 推奨する構成

```text
DesignForTools
├─ apps
│  ├─ web
│  └─ desktop
├─ crates
│  ├─ rust-core
│  ├─ node-engine
│  ├─ pose-engine
│  ├─ sprite-engine
│  └─ workflow-engine
├─ packages
│  ├─ ui
│  └─ schemas
└─ docs
```

この構成では、`rust-core` がアプリケーション全体の共有ロジックを担います。

将来的に次のプラットフォームへ展開する場合でも、ノード実行やポーズ補正などのロジックを共通利用できます。

- Web版
- Tauri版
- モバイル版
- クラウド版

分離された Rust Core は、長期運用においてテスト、保守、再利用の拡張性を高めます。
