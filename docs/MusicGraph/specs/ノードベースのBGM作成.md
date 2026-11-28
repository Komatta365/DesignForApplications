# ノードベースのBGM作成

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
    background-color: #f5f5f5;
}
</style>

ノードベースのBGM作成では、楽曲を小さな生成・処理要素に分割し、ノードを接続して組み立てます。生成AIが画像から直接楽曲を作る方式ではなく、コード進行、メロディ、伴奏、音源、ミックスなどを独立して制御できる構成です。

## 基本ワークフロー

```text
[テーマ]
    ↓
[コード進行]
    ↓
[メロディ]
    ↓
[アレンジ]
    ↓
[楽器選択]
    ↓
[MIDI出力]
    ↓
[音源レンダリング]
    ↓
[完成BGM]
```

基本的なノードは、入力となる楽曲パラメータ、音楽素材を生成するノード、音声を処理するノード、最終的な出力ノードに分けます。

## ノード一覧

### 1. 楽曲設計ノード

| ノード | 役割 | 例 |
|------|------|----|
| Genre | ジャンルを決定する | Fantasy / Rock / Jazz / EDM / Orchestra / Ambient |
| Mood | 曲の雰囲気を決定する | 明るい / 悲しい / 壮大 / 不気味 / 緊張感 / 癒し |
| BPM | テンポを設定する | 60～200 |
| Key | 調を設定する | C Major / A Minor / G Major |

これらのノードは、後続の生成ノードへ共通の楽曲条件を渡します。

### 2. Chord Generator

ジャンル、雰囲気、調からコード進行を生成します。

```text
入力
├─ Genre
├─ Mood
└─ Key

出力
└─ C - G - Am - F
   Dm - G - C - Am
```

プリセットとして、以下の進行を扱います。

- J-POP進行
- カノン進行
- 丸サ進行
- 王道進行

### 3. Melody Generator

コード進行、BPM、雰囲気からメロディを生成します。

```text
入力
├─ Chord
├─ BPM
└─ Mood

出力
└─ MIDI Note列
```

生成方式は、用途や実装段階に応じて切り替えます。

- Markov モデル
- Transformer ベースのMIDI生成
- MusicGen などの音楽生成モデル

### 4. Bass Generator

コード進行からベースパートを生成します。

```text
入力
└─ Chord

出力
└─ Bass MIDI
```

対応するパターン：

- Root
- Walking Bass
- Arpeggio

### 5. Drum Generator

ジャンルとBPMからドラムパートを生成します。

```text
入力
├─ Genre
└─ BPM

出力
└─ Drum MIDI
```

プリセット：

- Rock
- EDM
- House
- Lo-Fi

### 6. Arpeggiator

コードからアルペジオを生成します。

```text
入力
└─ Chord

出力
└─ Arpeggio MIDI
```

再生モード：

- Up
- Down
- Random
- PingPong

### 7. 楽器ノード

MIDIを、指定した音源でレンダリングします。

| 楽器ノード | 入力 | 出力 |
|----------|------|------|
| Piano | MIDI | Audio |
| Strings | MIDI | Audio |
| Brass | MIDI | Audio |
| Guitar | MIDI | Audio |
| Synth | MIDI | Audio |
| Drums | MIDI | Audio |
| Drum Kit | MIDI | Audio |

### 8. Mixer

複数の音声トラックをまとめて、マスター音声を作ります。

```text
Track 1 ┐
Track 2 ┼→ [Mixer] → [Mix Audio]
Track 3 ┘
```

主なパラメータ：

- Volume
- Pan
- Mute
- Solo

### 9. エフェクトノード

音声を別の音声へ変換するノードです。

| ノード | 主な設定 |
|--------|----------|
| Reverb | Room / Hall / Plate |
| Delay | Delay time / Feedback |
| Compressor | Threshold / Ratio |
| EQ | Frequency / Gain / Q |

### 10. Mastering / Export

最終的な音質調整とファイル出力を行います。

```text
[Mix Audio]
    ↓
[Limiter / EQ / Multiband Compressor]
    ↓
[Export]
    ↓
[WAV / MP3 / OGG / MIDI]
```

## MVP

最初のMVPでは、AI生成より先に、決定的なMIDI生成と出力の検証を行います。

```text
[BPM]
    ↓
[Key / Genre / Mood]
    ↓
[Chord Generator]
    ├→ [Melody Generator] → [MIDI]
    ├→ [Bass Generator]   → [MIDI]
    └→ [Drum Generator]   → [MIDI]
```

MVPの対象ノード：

| 分類 | ノード |
|------|--------|
| 基本設定 | BPM / Key / Genre / Mood |
| 音楽生成 | Chord Generator / Melody Generator / Bass Generator / Drum Generator / Arpeggiator |
| 音源 | Piano / Strings / Synth / Drum Kit |
| 処理 | Reverb / Delay / EQ / Mixer |
| 出力 | WAV / MP3 / OGG / MIDI |

優先順位は以下とします。

1. コード進行を生成する
2. コード進行からメロディを生成する
3. MIDIファイルへ出力する
4. 音源でMIDIをレンダリングする
5. ミックス、エフェクト、マスタリングを追加する
6. 画像解析とAI生成を統合する

## AIとの連携

画像解析や文章入力で楽曲条件を作り、既存のMusicGraphノードへ接続できます。

```text
画像
  ↓
CLIP
  ↓
Scene Analyzer
  ↓
Emotion Extractor
  ↓
MusicGraph Parameter
  ↓
Music Nodes
  ↓
完成BGM
```

別案として、文章プロンプトから同じパラメータを生成する構成も利用できます。

```text
AIプロンプト
  ↓
Genre / Mood / BPM / Instruments
  ↓
BGM生成
```

### 例：森の画像から生成

```text
森の画像
  ↓
Scene Analysis
├─ Genre = Fantasy
├─ Mood = Calm
├─ BPM = 80
└─ Instruments = Flute + Strings
  ↓
自動BGM生成
```

### ゲームイベントとの連携

```text
ゲームイベント
├─ Exploration
├─ Battle
├─ Town
└─ Victory
      ↓
対応するMusicGraphワークフロー
      ↓
シーン用BGM
```

感情マップをノードグラフまたはタイムラインに沿って定義することで、楽曲を自動遷移させます。

```text
Calm → Tension → Battle → Victory
```

## KMusicGraph コンセプト

KMusicGraphは、音楽版のComfyUIを目指すノードベース音楽制作環境です。

```text
Node-Based Music Creation Platform

画像 → 感情解析 → BGM生成
ノード接続 → BGM生成
AIプロンプト → BGM生成
```

目標は、専門的な作曲知識がなくても、誰でもゲーム用BGMを作れる仕組みを提供することです。

### システム構成

```text
KMusicGraph
├─ Frontend
│  ├─ React
│  ├─ ReactFlow
│  └─ Zustand
├─ Rust Core
│  ├─ MIDI Engine
│  ├─ Harmony Engine
│  ├─ Melody Engine
│  ├─ Audio Engine
│  └─ Plugin Runtime
├─ AI
│  ├─ CLIP
│  ├─ ONNX Runtime
│  ├─ MusicGen
│  └─ LLM
└─ Assets
   ├─ SoundFont
   ├─ Samples
   └─ Presets
```

### TypeScript + Rust + Tauri

推奨する責務分担は以下のとおりです。

| レイヤー | 技術 | 主な責務 |
|----------|------|----------|
| UI | TypeScript / React / ReactFlow | ノード編集、接続管理、プレビュー |
| 状態管理 | Zustand | ノードパラメータ、ワークフロー状態 |
| 音楽処理 | Rust Core | コード進行、メロディ、MIDI、音声処理 |
| デスクトップ連携 | Tauri | ファイル出力、OS連携、ローカルAI実行 |
| AI推論 | ONNX Runtime / CLIP / MusicGen | 画像解析、感情抽出、音楽生成 |

### リポジトリ案

```text
KMusicGraph
├─ apps
│  ├─ desktop
│  └─ web
├─ crates
│  ├─ kmusic-core
│  ├─ kmusic-midi
│  ├─ kmusic-audio
│  ├─ kmusic-ai
│  └─ kmusic-plugin
└─ packages
   ├─ ui
   ├─ node-system
   └─ workflow
```

## プラグイン構想

将来的にはノードをプラグインとして追加できる構成にします。

```text
plugins/
├─ musicgen/
├─ stable-audio/
├─ suno/
├─ openvino/
└─ vst/
```

ワークフローはノードとリンクを含むJSONとして保存・共有します。

```json
{
  "nodes": [],
  "links": []
}
```

## 独自性

通常のDAWとは異なり、KMusicGraphは次の入力からゲーム用BGMを構築することを重視します。

- 画像からBGMを生成する
- ゲームイベントからBGMを生成する
- 感情マップに沿って楽曲を遷移する
- ノード構成を共有して再利用する
- 生成AIノードと従来のMIDI処理を組み合わせる

まずは ReactFlow + Rust Core + MIDI出力のMVPを完成させ、その後に画像解析、MusicGen、外部音楽生成モデルを追加する構成が推奨されます。

## 関連ドキュメント

- [MusicGraph](../index.md)
- [メロディーの改善](./メロディーの改善.md)
- [現実装の評価](./現実装の評価.md)
- [Composition層のノード構成](./Composition層のノード構成.md)
- [ノードごとの入出力型](./ノードごとの入出力型.md)
- [TypeScriptの型定義](./TypeScriptの型定義.md)
- [TypeScript + Rust + Tauri ハイブリッドアーキテクチャ](../../typescript-react-rust-wasm-tauri/index.md)
