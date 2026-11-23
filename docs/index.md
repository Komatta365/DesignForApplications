# ドキュメント

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

ドキュメント作成と整備の進捗状況：

- [x] メインドキュメント構成（docs/index.md）
- [x] TypeScript + Rust + Tauri ハイブリッドアーキテクチャ ガイド
- [x] Rust Core 設計パターン
- [x] Web版実装 ガイド
- [x] 2026年標準 Webゲーム開発ガイド
- [x] 2DSpriteGraph ドキュメント
- [x] SFXGraph ドキュメント
- [x] MusicGraph ドキュメント
- [x] ActionGameEditor ドキュメント
- [ ] 実装コード例の充実化
- [ ] API リファレンス
- [ ] トラブルシューティングガイド
- [ ] チュートリアルビデオ制作

## 目次

- [ドキュメント構成](#ドキュメント構成)
  - [TypeScript (React) + Rust Core + WebAssembly + Tauri](#typescript-react--rust-core--webassembly--tauri)
  - [ゲーム開発・ツール](#ゲーム開発ツール)
    - [2DSpriteGraph](#2dspritegraph)
    - [SFXGraph](#sfxgraph)
    - [MusicGraph](#musicgraph)
    - [ActionGameEditor](#actiongameeditor)

---

## ドキュメント構成

### [TypeScript (React) + Rust Core + WebAssembly + Tauri](typescript-react-rust-wasm-tauri/)

ハイブリッドアプリケーション開発のための総合ガイド

#### 関連ドキュメント

- [Rust Core](typescript-react-rust-wasm-tauri/RustCore.md) - アプリケーション中核ロジックの設計パターン
- [Webブラウザーで動作させる最適解](typescript-react-rust-wasm-tauri/WebBrowser.md) - Web版とDesktop版の共有アーキテクチャ
- [Webブラウザでのゲーム開発](typescript-react-rust-wasm-tauri/WebBrowserGameDevelopment.md) - 2026年標準のゲーム開発ガイド（WebGPU、AI統合、パフォーマンス最適化）

---

## ゲーム開発・ツール

### [2DSpriteGraph](2DSpriteGraph/)

2Dスプライト画像を効率的に管理・編集するノードベースグラフエディタ

**主な機能:**
- スプライト管理と変換処理
- アニメーション定義
- スプライトシート生成
- メタデータ出力

### [SFXGraph](SFXGraph/)

ゲーム用の効果音（SFX）を管理・処理するノードベースオーディオエディタ

**主な機能:**
- 音声ファイル管理
- エフェクト処理（リバーブ、EQ、コンプレッサーなど）
- 波形解析と最適化
- バッチ処理対応

### [MusicGraph](MusicGraph/)

ゲーム用BGMと楽曲を管理・編集するノードベースミュージックエディタ

**主な機能:**
- 楽曲ファイル管理
- テンポ・ピッチ調整
- ループ・シーン管理
- 適応型音楽システム対応

### [ActionGameEditor](ActionGameEditor/)

アクションゲームの制作を支援する統合エディタ

**主な機能:**
- ゲームロジック設計
- キャラクター制御
- 敵AI制御
- 状態管理と衝突判定
