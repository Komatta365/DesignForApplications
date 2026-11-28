# MusicGraph

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

MusicGraph は、ゲーム用のBGMや楽曲を効率的に管理・編集するためのノードベースミュージックエディタです。

## TODO リスト

MusicGraph 開発の進捗：

- [x] アーキテクチャ設計
- [x] ノードベースUI
- [x] 楽曲ファイル管理
- [x] テンポ・ピッチ調整
- [x] ループ・シーン管理
- [x] 適応型音楽システム設計
- [ ] テンポシンクロナイザー実装
- [ ] インタラクティブレイヤーシステム
- [ ] 動的シーン遷移
- [ ] オーケストレーション機能
- [ ] リアルタイム再生エンジン
- [ ] Advanced メモリ管理

## 目次

- [概要](#概要)
- [ノードベースのBGM作成](#ノードベースのbgm作成)
- [メロディーの改善](#メロディーの改善)
- [現実装の評価](#現実装の評価)
- [Composition層のノード構成](#composition層のノード構成)
- [ノードごとの入出力型](#ノードごとの入出力型)
- [TypeScriptの型定義](#typescriptの型定義)
- [主な機能](#主な機能)
- [音楽処理ノード](#音楽処理ノード)
- [使用例](#使用例)
- [出力形式](#出力形式)
- [適応型音楽システム](#適応型音楽システム)
- [推奨される用途](#推奨される用途)
- [関連ドキュメント](#関連ドキュメント)

---

## 概要

MusicGraph は、以下の機能を提供します：

- **ノードベースミュージックエディタ** - BGM管理と処理
- **楽曲ファイル管理** - 複数の音楽ファイルを統一的に管理
- **ループ・シーン処理** - ゲームシーンに応じた音楽フェード・切り替え
- **メタデータ出力** - JSON形式での楽曲情報エクスポート
- **適応型音楽対応** - インタラクティブなBGM設定

## ノードベースのBGM作成

コード進行、メロディ、ベース、ドラム、楽器、ミキサーなどをノードとして接続し、MIDIやゲーム用BGMを生成する設計です。

```text
テーマ → コード進行 → メロディ → アレンジ → 楽器 → MIDI → 音声出力
```

AIによる画像解析や感情抽出も、既存ノードへのパラメータ入力として統合できます。

詳細は「[ノードベースのBGM作成](specs/ノードベースのBGM作成.md)」を参照してください。

## メロディーの改善

AIが生成したメロディーが単調になる問題に対して、Motif、Phrase、Counter Melody、Variation、Humanizerへ工程を分割します。

```text
モチーフ生成 → フレーズ展開 → 対旋律 → 変奏 → 演奏表現 → MIDI
```

感情カーブとフレーズライブラリを組み合わせ、AIには抽象的な作曲条件を生成させ、実際の音符構築はRust Coreで行います。

詳細は「[メロディーの改善](specs/メロディーの改善.md)」を参照してください。

## 現実装の評価

現在の設計をインタラクティブミュージックエンジンとして評価し、Composition層を改善する方針を整理しています。

特にMotif、Phrase、Variation、Counter Melody、Transitionを追加し、ゲームParameterからコード進行や楽曲構造を直接制御する構成を重視します。

詳細は「[現実装の評価](specs/現実装の評価.md)」を参照してください。

## Composition層のノード構成

Composition層をMotif、Phrase、Variation、Counter Melody、Transitionなどの12ノードへ分解し、AIによる単調なメロディ生成ではなく、再現性のある作曲工程を構築します。

```text
Motif → Phrase → Variation → Counter Melody → Arrangement → Performance
```

Adaptive MelodyによってゲームParameterを音数、跳躍、装飾、Transitionへ直接接続する構成です。

詳細は「[Composition層のノード構成](specs/Composition層のノード構成.md)」を参照してください。

## ノードごとの入出力型

ComfyUIと同様の型付きポートを採用し、SongData、PhraseData、NoteSequence、AudioDataなどの接続互換性を明示します。

```text
Theory → Composition → NoteSequence → Instrument → AudioData
ParameterData → EnergyData → Composition
```

ゲーム状態を作曲そのものへ反映するため、MoodData、EnergyData、ArrangementData、TransitionDataも共通型として定義します。

詳細は「[ノードごとの入出力型](specs/ノードごとの入出力型.md)」を参照してください。

## TypeScriptの型定義

`@kmusic/types`として共有するドメインモデルを定義し、Timeline、Theory、Composition、MIDI、Audio、Parameter、Workflowを一つの型体系で扱います。

```typescript
MusicIntent → MotifData → PhraseData → NoteSequence → AudioData
```

ReactFlow向けのPortDefinition、KNode、KMusicGraphと、AI ComposerからRust Coreへ渡すMusicIntentも含めます。

詳細は「[TypeScriptの型定義](specs/TypeScriptの型定義.md)」を参照してください。

## 主な機能

### 音楽ファイル入力

ゲーム用BGMのインポート：

```
入力ノード
├─ WAV/MP3/OGG対応
├─ ループポイント設定
├─ メタデータ抽出
└─ 長さ・テンポ自動検出
```

### ループ・シーン管理

ゲームシーンに応じた音楽制御：

```
シーン管理ノード
├─ ループ設定
├─ シーン遷移フェード
├─ インタラクティブレイヤー
└─ 条件分岐
```

### 音質・テンポ調整

楽曲のカスタマイズ：

```
パラメータノード
├─ テンポ変更（BPM）
├─ ピッチシフト
├─ ゲイン（音量）
├─ イコライザー（EQ）
└─ エフェクト処理
```

### 波形解析と最適化

楽曲の特性解析：

```
解析ノード
├─ ピークレベル検出
├─ BPM自動検出
├─ ジャンル判別
└─ 自動ノーマライズ
```

## 音楽処理ノード

### 入力ノード

```json
{
  "type": "input",
  "file": "boss_theme.mp3",
  "sampleRate": 44100,
  "bitRate": 320,
  "duration": 180.5,
  "bpm": 140,
  "loopStart": 10.0,
  "loopEnd": 170.0
}
```

### テンポ調整ノード

```json
{
  "type": "tempoShift",
  "originalBpm": 140,
  "targetBpm": 160,
  "preservePitch": true,
  "quality": "high"
}
```

### ピッチシフトノード

```json
{
  "type": "pitchShift",
  "semitones": 2,
  "preserveTempo": true,
  "quality": "high"
}
```

### イコライザーノード

```json
{
  "type": "equalizer",
  "bands": [
    {
      "frequency": 80,
      "gain": -3,
      "Q": 0.7,
      "type": "lowshelf"
    },
    {
      "frequency": 1000,
      "gain": 2,
      "Q": 1.0,
      "type": "peaking"
    },
    {
      "frequency": 10000,
      "gain": 4,
      "Q": 1.0,
      "type": "highshelf"
    }
  ]
}
```

### シーン遷移ノード

```json
{
  "type": "sceneTransition",
  "fromScene": "exploration",
  "toScene": "battle",
  "fadeOutDuration": 2.0,
  "fadeInDuration": 1.5,
  "crossfade": true
}
```

## 使用例

### シンプルなBGM処理

```
音楽読込 → ループポイント設定 → ゲイン調整 → MP3出力
```

### インタラクティブなBGM

```
戦闘BGM読込
    ↓
強度レイヤー定義
    ↓
シーン遷移フェード設定
    ↓
メタデータ生成
    ↓
出力
```

### 複雑な音楽処理パイプライン

```
複数BGM読込
    ↓
個別テンポ・ピッチ調整
    ↓
イコライザー処理
    ↓
ループ最適化
    ↓
シーン遷移マップ設定
    ↓
メタデータ生成
    ↓
出力
```

## 出力形式

### 処理済み楽曲（MP3/OGG）

圧縮形式で出力：

```json
{
  "format": "mp3",
  "bitRate": 192,
  "sampleRate": 44100,
  "channels": 2
}
```

### メタデータ（JSON）

```json
{
  "music": [
    {
      "id": "exploration",
      "file": "exploration_theme.mp3",
      "duration": 180.5,
      "category": "exploration",
      "volume": 1.0,
      "loopStart": 10.0,
      "loopEnd": 170.0,
      "loopEnabled": true,
      "tags": ["nature", "peaceful"]
    },
    {
      "id": "boss_battle",
      "file": "boss_theme.mp3",
      "duration": 240.0,
      "category": "battle",
      "volume": 1.0,
      "loopStart": 15.0,
      "loopEnd": 220.0,
      "loopEnabled": true,
      "tags": ["intense", "dramatic"]
    }
  ]
}
```

### シーン遷移マップ（JSON）

```json
{
  "sceneMusic": {
    "menu": {
      "file": "menu_theme.mp3",
      "fadeTime": 1.0
    },
    "exploration": {
      "file": "exploration_theme.mp3",
      "fadeTime": 2.0
    },
    "battle": {
      "file": "boss_theme.mp3",
      "fadeTime": 1.5
    }
  },
  "transitions": [
    {
      "from": "exploration",
      "to": "battle",
      "fadeOutTime": 2.0,
      "fadeInTime": 1.5,
      "type": "crossfade"
    },
    {
      "from": "battle",
      "to": "exploration",
      "fadeOutTime": 3.0,
      "fadeInTime": 2.0,
      "type": "fadethrough"
    }
  ]
}
```

## 適応型音楽システム

ゲームの状態に応じた動的なBGM制御：

### インテンシティレイヤー

```json
{
  "adaptiveMusic": {
    "baseTheme": "battle_theme.mp3",
    "layers": [
      {
        "name": "calm",
        "intensity": 0.0,
        "file": "battle_calm.mp3"
      },
      {
        "name": "normal",
        "intensity": 0.5,
        "file": "battle_normal.mp3"
      },
      {
        "name": "intense",
        "intensity": 1.0,
        "file": "battle_intense.mp3"
      }
    ]
  }
}
```

### 状態遷移マップ

```json
{
  "stateMachine": {
    "exploration": {
      "music": "exploration_theme",
      "transitions": {
        "encounter_weak": {
          "targetState": "battle_easy",
          "fadeTime": 1.0
        },
        "encounter_boss": {
          "targetState": "battle_intense",
          "fadeTime": 2.0
        }
      }
    },
    "battle_easy": {
      "music": "battle_theme_easy",
      "adaptiveIntensity": true,
      "transitions": {
        "victory": {
          "targetState": "exploration",
          "fadeTime": 2.0
        }
      }
    }
  }
}
```

## 推奨される用途

| 用途 | 説明 |
|------|------|
| ゲームBGM管理 | すべてのBGMを統一的に処理・出力 |
| 音楽ライブラリ構築 | ゲーム用の楽曲ライブラリを自動生成 |
| シーン遷移音楽 | シーンの切り替えに応じた音楽フェード管理 |
| 適応型音楽設定 | ゲームの状態に応じた動的BGM制御 |
| 音質最適化 | 配布用に楽曲を自動最適化 |

## 関連ドキュメント

- [ノードベースのBGM作成](specs/ノードベースのBGM作成.md)
- [メロディーの改善](specs/メロディーの改善.md)
- [現実装の評価](specs/現実装の評価.md)
- [Composition層のノード構成](specs/Composition層のノード構成.md)
- [ノードごとの入出力型](specs/ノードごとの入出力型.md)
- [TypeScriptの型定義](specs/TypeScriptの型定義.md)
- [DesignForApplications 総合ガイド](../index.md)
- [TypeScript + Rust + Tauri ハイブリッドアーキテクチャ](../typescript-react-rust-wasm-tauri/index.md)
- [2DSpriteGraph - スプライト管理](../2DSpriteGraph/index.md)
- [SFXGraph - 効果音管理](../SFXGraph/index.md)
- [ActionGameEditor - アクションゲーム制作](../ActionGameEditor/index.md)
