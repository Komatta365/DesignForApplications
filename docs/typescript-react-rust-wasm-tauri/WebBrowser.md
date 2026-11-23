# Webブラウザーで動作させる最適解

<style>
a {
    text-decoration: none;
    color: #0066ff;
}
a:hover {
    text-decoration: underline;
}
</style>

## 目次

- [全体構成](#全体構成)
- [なぜこれが最適か](#なぜこれが最適か)
  - [ノード実行エンジンを共有できる](#ノード実行エンジンを共有できる)
  - [Web版が作りやすい](#web版が作りやすい)
  - [Tauri版も同じコアを利用](#tauri版も同じコアを利用)
- [やってはいけない構成](#やってはいけない構成)
- [プロジェクト構成](#プロジェクト構成)
- [結論](#結論)
- [関連ドキュメント](#関連ドキュメント)

---

## 全体構成

TypeScript + Rust + Tauri でノードベースAI制作ツールを開発する場合、Webブラウザー対応の最適解は次の構成です。

```text
Frontend
  React + TypeScript

Core
  Rust

Web
  WebAssembly (WASM)

Desktop
  Tauri

Server (Optional)
  Axum
```

全体像は、Rust Core を共有ロジックとして Web、デスクトップ、サーバーの各ランタイムから利用する構成です。

```text
                  ┌───────────┐
                  │ Rust Core │
                  └─────┬─────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
        ▼               ▼               ▼
       WASM            Tauri          Axum API
        │               │               │
        ▼               ▼               ▼
       Web           Desktop           Cloud
```

## なぜこれが最適か

### ノード実行エンジンを共有できる

次の処理をすべて Rust Core に配置します。

- `Node`
- `Workflow`
- `Loop`
- JSON Schema
- ポーズ処理
- スプライト補正

これにより、Web版、デスクトップ版、サーバー版が同じロジックを利用できます。

```text
Web版
Desktop版
Server版
  ↓
同じ Rust Core
```

### Web版が作りやすい

Rust Core を WebAssembly にビルドします。

```shell
wasm-pack build
```

TypeScriptからは、次のように呼び出せます。

```typescript
import { executeWorkflow } from "rust-core";

executeWorkflow(workflow);
```

ブラウザーだけでも次の機能を実装できます。

- ノード編集
- ノード実行
- 軽量画像処理

### Tauri版も同じコアを利用

重い処理だけをTauri側で実行します。

- GPUアクセス
- ローカルファイル
- FFmpeg
- AIモデル

結果として、次の2段階の機能構成になります。

```text
Web版
  軽量機能

Desktop版
  フル機能
```

この構成は、FigmaやComfyUIのような「軽量なWeb版と高機能なデスクトップ版」の併用に近い設計です。

## やってはいけない構成

最初からReactからTauriコマンドへ直接依存する設計は避けます。

```text
React
  ↓
Tauri Command
  ↓
Rust
```

たとえば、次のようにTauri固有の関数だけで設計すると後からWeb版を追加する際に全面改修が必要になります。

```rust
#[tauri::command]
fn execute_workflow(...) {}
```

代わりに、Tauri固有処理はUIとRust Coreの境界に分離し、共有ロジックをプラットフォームから独立させます。

## プロジェクト構成

モノレポ構成を推奨します。

```text
DesignForTools/
├─ apps/
│  ├─ web/
│  └─ desktop/
├─ crates/
│  ├─ rust-core/
│  ├─ node-engine/
│  ├─ workflow-engine/
│  ├─ pose-engine/
│  └─ sprite-engine/
├─ packages/
│  ├─ ui/
│  └─ schemas/
└─ docs/
```

この構成なら、将来次のプラットフォームへ拡張しやすくなります。

- Web版
- Tauri版
- Android版
- iOS版
- クラウド版

## 結論

2026年以降も見据える場合は、「Rust Core → WASM + Tauri」の二本立てが有望です。

Tauri中心ではなく、Rust Core中心で設計することで、ノード実行、ワークフロー、ポーズ補正などのロジックを長期的に共有できます。

---

## 関連ドキュメント

ゲーム開発など、特定ユースケースの実装詳細については、以下のドキュメントを参照してください。

- [Webブラウザでのゲーム開発](WebBrowserGameDevelopment.md) - WebGPU、AI統合、パフォーマンス最適化を含むゲーム開発ガイド
