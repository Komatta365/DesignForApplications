# TypeScript (React) + Rust Core + WebAssembly + Tauri

<style>
a {
    text-decoration: none;
    color: #90CAF9;
}
a:hover {
    text-decoration: underline;
    color: #FFFFFF;
}
tr th, tr td {
    border: 1px solid #e6e6e6;
}
tr th {
    background-color: #e9efff;
    color: #1f2328;
    font-weight: 700;
}
</style>

## TODO リスト

実装進捗状況：

- [x] 全体アーキテクチャ設計
- [x] TypeScript/React セットアップ
- [x] Rust Core 実装パターン
- [x] WASM 連携方式
- [x] Tauri 統合
- [x] エラーハンドリング設計
- [x] セキュリティ設計
- [x] テスト戦略
- [ ] 実装コード例（完全版）
- [ ] デプロイ自動化
- [ ] パフォーマンスチューニングガイド
- [ ] トラブルシューティングガイド
- [ ] ベストプラクティス集
  - [ ] ベストプラクティス集2
  - [ ] ベストプラクティス集3
    - [ ] ベストプラクティス集4

## 目次

- [関連ドキュメント](#関連ドキュメント)
- [1. 概要](#1-概要)
- [2. なぜ組み合わせるのか](#2-なぜ組み合わせるのか)
  - [2.1 TypeScript / React](#typescript--react)
  - [2.2 Rust](#rust)
  - [2.3 WebAssembly](#webassembly)
  - [2.4 Tauri](#tauri)
- [3. 推奨アーキテクチャ](#3-推奨アーキテクチャ)
- [4. プロジェクト構成例](#4-プロジェクト構成例)
- [5. Rust Core の実装例](#5-rust-core-の実装例)
- [6. WASM 連携](#6-wasm-連携)
- [7. Tauri連携](#7-tauri連携)
- [8. 実行環境の切り替え](#8-実行環境の切り替え)
- [9. データフロー](#9-データフロー)
  - [9.1 ブラウザ版](#ブラウザ版)
  - [9.2 Tauri版](#tauri版)
- [10. エラー設計](#10-エラー設計)
- [11. セキュリティ](#11-セキュリティ)
- [12. テスト戦略](#12-テスト戦略)
  - [12.1 Rust Core](#rust-core)
  - [12.2 WASM](#wasm)
  - [12.3 Tauri](#tauri)
  - [12.4 React](#react)
- [13. 開発手順](#13-開発手順)
  - [13.1 必要なツール](#必要なツール)
  - [13.2 初期構築例](#初期構築例)
  - [13.3 開発サーバー](#開発サーバー)
- [14. 配布](#14-配布)
- [15. 推奨事項](#15-推奨事項)
- [16. まとめ](#16-まとめ)

---

## 関連ドキュメント

- [Rust Core](RustCore.md)
- [Webブラウザーで動作させる最適解](WebBrowser.md)
- [Webブラウザでのゲーム開発](WebBrowserGameDevelopment.md)

## 1. 概要

本構成は、UI に React / TypeScript、計算・ビジネスロジックの中核に Rust、ブラウザ互換コードに WebAssembly（WASM）、ネイティブデスクトップ機能に Tauri を使用するハイブリッドアーキテクチャです。

各技術の役割を次のように分離します。

| レイヤー           | 技術               | 主な役割                                           |
| ------------------ | ------------------ | -------------------------------------------------- |
| プレゼンテーション | React + TypeScript | 画面描画、ユーザー操作、状態管理                   |
| アプリケーション   | Tauri              | ネイティブ API、ウィンドウ、ファイル、プロセス連携 |
| コアロジック       | Rust               | 安全で高速な処理、ドメインモデル、共有ライブラリ   |
| ブラウザ配布       | WebAssembly        | Rust コアをブラウザ上で実行                        |
| ネイティブ配布     | Tauri              | Rust コマンドをデスクトップアプリとして実行        |

Rust コードを二重管理せず、デスクトップとブラウザで同じコアロジックを利用できる点が本構成の中心です。

## 2. なぜ組み合わせるのか

### TypeScript / React

- DOM または WebView を使ったリッチな UI を短期間で構築できる
- コンポーネント分割、状態管理、ルーティング、テストのエコシステムが充実している
- WASM や Tauri の境界を意識しつつ、画面ロジックを記述できる

### Rust

- メモリ安全性と型安全性が高い
- 高負荷な計算、並列処理、バイナリ解析などに適する
- ネイティブ実行と WASM ターゲットの両方をサポートする

### WebAssembly

- ブラウザ上で Rust コアをほぼネイティブに近い速度で実行できる
- JavaScript / TypeScript と相互運用できる
- Web版、デモ、エディター機能などを提供できる

### Tauri

- 軽量なネイティブデスクトップアプリを構築できる
- Rust 側でファイルシステム、OS API、システムプロセスなどを扱える
- 配布時にブラウザ向けランタイムを別途用意する必要がない

## 3. 推奨アーキテクチャ

```text
TypeScript / React
├── Components / Views
├── Application Services
├── Platform Adapter
│   ├── Tauri Commands
│   └── WASM Bindings
└── Shared TypeScript Types

Rust Core
├── Domain Models
├── Pure Functions
├── Validation
└── Processing / Algorithms

Rust Platform
├── Tauri Commands
├── File System
└── OS Integration
```

依存関係は原則として次の方向にします。

```text
React → Application Service → Platform Adapter
                            ├── Tauri Bridge
                            └── WASM Bridge

Tauri Bridge  → Rust Core
WASM Bridge   → Rust Core
```

React コンポーネントから Rust を直接呼び出さず、アプリケーションサービスとプラットフォームアダプター経由にすると、ブラウザ版と Tauri版の切り替えが容易になります。

## 4. プロジェクト構成例

```text
project/
├── src/
│   ├── components/
│   ├── services/
│   ├── adapters/
│   │   ├── tauri.ts
│   │   └── wasm.ts
│   ├── types/
│   └── main.tsx
├── src-tauri/
│   ├── src/
│   │   ├── main.rs
│   │   └── commands.rs
│   ├── Cargo.toml
│   └── tauri.conf.json
├── core/
│   ├── src/
│   │   └── lib.rs
│   └── Cargo.toml
├── package.json
└── vite.config.ts
```

Rust コアは Tauri アプリケーションとは別のクレートとして管理することを推奨します。

```text
workspace/
├── Cargo.toml
├── core/
└── src-tauri/
```

ワークスペース例:

```toml
[workspace]
members = ["core", "src-tauri"]
resolver = "2"
```

## 5. Rust Core の実装例

WASM と Tauri の両方で利用できる純粋な Rust コアを定義します。

```rust
// core/src/lib.rs
pub fn normalize_name(input: &str) -> Result<String, String> {
    let value = input.trim();

    if value.is_empty() {
        return Err("名前を入力してください".to_string());
    }

    Ok(value.chars().take(100).collect())
}
```

```rust
// core/src/lib.rs
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn rejects_empty_name() {
        assert!(normalize_name(" ").is_err());
    }

    #[test]
    fn normalizes_name() {
        assert_eq!(normalize_name("  App  ").unwrap(), "App");
    }
}
```

Rust Core は OS APIやブラウザ APIを直接参照せず、入力と出力を明確にした純粋処理を中心にします。

## 6. WASM 連携

WASM 版では `wasm-bindgen` を使って JavaScript から Rust を呼び出します。

```toml
[dependencies]
wasm-bindgen = "0.2"
```

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn normalize_name(input: &str) -> Result<String, JsValue> {
    core::normalize_name(input).map_err(|message| JsValue::from_str(&message))
}
```

React 側のアダプター例:

```typescript
import init, { normalize_name } from "../pkg/core";

export async function normalizeName(input: string): Promise<string> {
  await init();

  try {
    return normalize_name(input);
  } catch (error) {
    throw new Error(String(error));
  }
}
```

ビルド例:

```powershell
rustup target add wasm32-unknown-unknown
cargo install wasm-pack
cd core
wasm-pack build --target web --out-dir ../pkg
```

実行時はブラウザ版の WASM初期化が完了してから関数を呼び出します。

## 7. Tauri連携

Tauri側で Rust Core を呼び、Reactから Tauriコマンドとして公開します。

```rust
// src-tauri/src/commands.rs
#[tauri::command]
pub fn normalize_name(input: String) -> Result<String, String> {
    core::normalize_name(&input)
}
```

```rust
// src-tauri/src/main.rs
mod commands;

fn main() {
    tauri::Builder::default()
        .invoke_handler(tauri::generate_handler![
            commands::normalize_name
        ])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

React側のアダプター例:

```typescript
import { invoke } from "@tauri-apps/api/core";

export async function normalizeName(input: string): Promise<string> {
  return invoke<string>("normalize_name", { input });
}
```

Reactコンポーネントは、実装詳細に依存しない共通インターフェイスを通して利用します。

```typescript
export interface NameNormalizer {
  normalize(input: string): Promise<string>;
}
```

## 8. 実行環境の切り替え

ブラウザ版と Tauri版を切り替える場合、環境判定は共通処理に閉じ込めます。

```typescript
import { invoke } from "@tauri-apps/api/core";

export function isTauri(): boolean {
  return "__TAURI_INTERNALS__" in window;
}

export async function normalizeName(input: string): Promise<string> {
  if (isTauri()) {
    return invoke<string>("normalize_name", { input });
  }

  const wasm = await import("../pkg/core");
  await wasm.default();
  return wasm.normalize_name(input);
}
```

環境判定を各コンポーネントへ分散させるとテストや保守が難しくなるため避けます。

## 9. データフロー

### ブラウザ版

```text
React UI
  → TypeScript Service
  → WASM Adapter
  → Rust Core
  → WASM Result
  → React State
```

### Tauri版

```text
React UI
  → TypeScript Service
  → Tauri Adapter
  → Tauri Command
  → Rust Core
  → IPC Result
  → React State
```

## 10. エラー設計

境界ごとにエラー形式を統一します。

| レイヤー           | 推奨形式                                   |
| ------------------ | ------------------------------------------ |
| Rust Core          | `Result<T, DomainError>`                   |
| Tauri Command      | シリアライズ可能な文字列または構造化エラー |
| WASM               | `Result<T, JsValue>` または専用エラー型    |
| TypeScript Service | 型付き例外または `Result<T, AppError>`     |
| React UI           | ユーザー向けメッセージとフィールドエラー   |

例外メッセージを画面から直接表示しないようにし、エラーコードと表示文を分離することを推奨します。

## 11. セキュリティ

TauriのIPCはフロントエンドから呼び出し可能な権限境界です。許可するコマンド、引数、アクセス先を最小限にします。

- ファイルパスはサンドボックス外への書き込みを制限する
- コマンド引数の型と長さを Rust側で検証する
- 任意のシェルコマンド文字列を受け取らない
- WebViewから任意の外部URLを読み込ませない
- 本番ビルドで不要な開発ツールやデバッグ機能を無効化する
- シークレットをフロントエンドへ埋め込まない
- TauriのCapability / Permissionを機能単位で必要最小限に設定する

## 12. テスト戦略

### Rust Core

- `cargo test`でドメインロジックと境界値を検証する
- ブラウザ機能やOS機能に依存しないテストを優先する

### WASM

- wasm-packでターゲットビルドを検証する
- Playwrightなどでブラウザ版の初期化と相互運用を検証する

### Tauri

- `tauri::command`の引数・結果をテストする
- デスクトップ版のファイル、ウィンドウ、OS連携を手動または自動テストする

### React

- Vitestなどでコンポーネントとアダプターをテストする
- アダプターはモックし、Rust実装ではなくアプリケーションロジックを検証する

推奨コマンド例:

```powershell
cargo test
npm test
npm run tauri dev
npm run tauri build
```

## 13. 開発手順

### 必要なツール

- Node.js
- npm または pnpm
- Rust
- cargo
- Tauri CLI
- wasm-pack
- プラットフォーム別の Tauriビルドツールチェーン

### 初期構築例

```powershell
npx create-tauri-app@latest
rustup target add wasm32-unknown-unknown
cargo install wasm-pack
cargo install tauri-cli
```

### 開発サーバー

```powershell
npm run tauri dev
```

WASM を利用する場合は、変更時の再ビルドまたは watch 実行を設定します。

## 14. 配布

| 配布形式   | 利用技術             |
| ---------- | -------------------- |
| ブラウザ版 | React + WASM         |
| Windows版  | React + Tauri + Rust |
| macOS版    | React + Tauri + Rust |
| Linux版    | React + Tauri + Rust |

デスクトップ版とブラウザ版で同じ Rust Coreを利用する場合も、シリアライズ形式、エラー形式、バージョン互換性を共通仕様として管理します。

## 15. 推奨事項

- Rust Coreには純粋なドメイン処理を置く
- Reactコンポーネントにはビジネス計算を置かない
- TauriとWASMの呼び出し部分をアダプターへ分離する
- Rust CoreとTypeScript型定義の整合性をテストする
- IPC境界で入力検証を必ず行う
- 大きなバイナリデータはコピーやシリアライズのコストを考慮する
- 開発版と本番版でCapabilityを分離する
- ビルド成果物、生成コード、依存キャッシュをリポジトリ管理対象外にする

## 16. まとめ

この構成では、画面をReact、ネイティブ機能をTauri、共有処理をRust Core、ブラウザ実行をWASMで担当させることで、複数プラットフォームで一貫した処理を提供できます。

最適な境界を設計するポイントは次のとおりです。

1. Rust Coreを環境に依存しない層として分離する
2. WASMとTauriをReactから隠すアダプターを作る
3. IPCやWASM境界で型とエラーを明確にする
4. 権限と外部入力を最小限かつ厳格に扱う
5. Rust Coreをテストし、プラットフォーム固有の結合テストで補完する
