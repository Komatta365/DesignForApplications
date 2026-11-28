# MusicGraph - TODO

MusicGraph は、ゲーム音楽制作のためのノードベースオーディオエディタです。

## 進捗状況

### 進行中 🔄

- [ ] GitHub Copilot CLI で生成する

### 予定 📋

- [ ] 一般的な歌を再現できる様にする
  - [ ] 猫ふんじゃった
  - [ ] シングルベル
  - [ ] ドラゴンぐエストのオープニング
  - [ ] J-POP
- [ ] サンプルを作成
  - [ ] asset/sample/以下にサンプルを作成
  - [ ] サンプルから作成メニューを追加する
- [ ] オーディオエフェクト実装
- [ ] リアルタイムプレビュー機能
- [ ] 複数トラック管理
- [ ] オートメーション機能
- [ ] ユーザーテスト実施

### Composition層の強化 ✨

specs/Composition層のノード構成.md、メロディーの改善.md で設計中

- [ ] モチーフからメロディーへのパイプライン構築
  - [ ] Motif ノード（2～4小節の短いフレーズ生成）
  - [ ] Phrase ノード（Motifを楽曲構造へ展開）
  - [ ] Variation ノード（移調、反行、逆行、リズム変化）
  - [ ] Counter Melody ノード（主旋律に対する対旋律生成）
  - [ ] Harmony Generator ノード（ChordからPad、Strings、Choirの和声生成）
- [ ] 高度なCompositionノード群
  - [ ] Sequence Generator（短いパターンを段階的に反復・移動）
  - [ ] Ostinato（同じリズムやパターンを繰り返す）
  - [ ] Ornament（装飾音を追加）
  - [ ] Call & Response（フレーズと応答フレーズの生成）
  - [ ] Transition（Section間を自然につなぐ）
  - [ ] Density Control（フレーズの音数を制御）
  - [ ] Adaptive Melody（ゲームParameterに応じて旋律を変化）

### ノードベースのBGM制作パイプライン ✨

specs/ノードベースのBGM作成.md で設計中

- [ ] 楽曲設計ノードの実装
  - [ ] Genre ノード（ジャンル設定：Fantasy、Rock、Jazz、EDM等）
  - [ ] Mood ノード（雰囲気設定：明るい、悲しい、壮大等）
  - [ ] BPM ノード（テンポ設定：60～200）
  - [ ] Key ノード（調設定：C Major、A Minor等）
- [ ] 生成AIとルールベース処理の役割分離
  - [ ] Chord Generator ノード（プリセット進行対応）
  - [ ] Melody Generator のスコープ明確化
  - [ ] Bass Generator ノードの実装
  - [ ] Drum Pattern Generator ノードの実装
- [ ] 完成パイプライン
  - [ ] MIDI出力への最適化
  - [ ] 音源レンダリング機能
  - [ ] 複数オーディオフォーマット出力（WAV、MP3、OGG）

### 型システムの統一 ✨

specs/TypeScriptの型定義.md で設計中

- [ ] @kmusic/types パッケージの実装
  - [ ] primitives.ts（基本型定義）
  - [ ] timeline.ts（タイムライン関連型）
  - [ ] theory.ts（音楽理論型）
  - [ ] composition.ts（作曲ノード型）
  - [ ] arrangement.ts（アレンジメント型）
  - [ ] midi.ts（MIDI型）
  - [ ] audio.ts（オーディオ型）
  - [ ] parameter.ts（パラメータ型）
  - [ ] node.ts（ノード基盤型）
  - [ ] workflow.ts（ワークフロー型）
  - [ ] intent.ts（作曲意図型）
- [ ] TypeScriptとRust連携の型境界明確化
- [ ] Workflow保存・復元のJSON化
- [ ] SFXGraph との型基盤共有

### 現実装の最適化 ✨

specs/現実装の評価.md に基づく改善

- [ ] Parameter層の強化（現在★★★★★★★ - 最高評価の維持）
  - [ ] ゲーム状態との連携強化
  - [ ] リアルタイムパラメータ制御
- [ ] Composition層の改善（現在★★★☆☆ → ★★★★★ を目指す）
  - [ ] メロディ生成の単調性を解決
  - [ ] モチーフ→フレーズ→変奏のパイプライン確立
- [ ] Timeline管理の最適化
  - [ ] セクション間のシームレス遷移
  - [ ] マーカー機能の拡張


### 完了 ✅

- [x] アーキテクチャ設計
- [x] ノードベースオーディオエディタUI

## 開発リンク

- [MusicGraph](./index.md)
- [Composition層のノード構成](./specs/Composition層のノード構成.md)
- [ノードベースのBGM作成](./specs/ノードベースのBGM作成.md)
- [メロディーの改善](./specs/メロディーの改善.md)
- [現実装の評価](./specs/現実装の評価.md)
- [TypeScriptの型定義](./specs/TypeScriptの型定義.md)
