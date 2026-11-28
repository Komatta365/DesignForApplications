# TypeScriptの型定義

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

K-Music-Graphでは、ノード名や任意の文字列ではなく、音楽制作の概念を表すドメインモデルを共有します。型定義を`@kmusic/types`として独立パッケージ化することで、UI、Workflow Engine、Rust連携、AI Composer、SFXGraphなどで共通利用できます。

目的：

- 型付きポートをコンパイル時に検証する
- TypeScriptとRust Coreのデータ境界を明確にする
- WorkflowをJSONとして保存・復元できるようにする
- AIが音符ではなく作曲指示を生成できるようにする
- K-SFX-Graphとノード基盤を共有できるようにする

## パッケージ構成

```text
packages/
└─ types/
   ├─ package.json
   ├─ tsconfig.json
   └─ src/
      ├─ primitives.ts
      ├─ timeline.ts
      ├─ theory.ts
      ├─ composition.ts
      ├─ arrangement.ts
      ├─ midi.ts
      ├─ audio.ts
      ├─ parameter.ts
      ├─ node.ts
      ├─ workflow.ts
      ├─ intent.ts
      └─ index.ts
```

## 基本型

```typescript
export type UUID = string;

export type PortType =
  | "SongData"
  | "SectionData"
  | "MarkerData"
  | "KeyData"
  | "ScaleData"
  | "ChordData"
  | "ChordProgressionData"
  | "MotifData"
  | "PhraseData"
  | "HarmonyData"
  | "RhythmData"
  | "ArrangementData"
  | "MoodData"
  | "EnergyData"
  | "TransitionData"
  | "NoteData"
  | "NoteSequence"
  | "MidiData"
  | "AudioData"
  | "ParameterData"
  | "TriggerData"
  | "Float"
  | "Bool";
```

`PortType`は接続可能なドメイン型を列挙します。設定画面用の文字列やNode IDはポート型には含めず、ノード定義やWorkflow管理の型として扱います。

## Timeline

```typescript
export interface SongData {
  bpm: number;
  timeSignature: [number, number];
  key: string;
  lengthBars: number;
  loop: boolean;
}

export interface SectionData {
  id: UUID;
  name: string;
  startBar: number;
  endBar: number;
}

export interface MarkerData {
  id: UUID;
  name: string;
  bar: number;
}
```

SongDataは楽曲全体、SectionDataはIntroやBattleなどの範囲、MarkerDataは1拍点または小節位置の出来事を表します。

## Theory

```typescript
export type Mode =
  | "Major"
  | "Minor"
  | "Dorian"
  | "Mixolydian"
  | "Lydian";

export interface KeyData {
  root: string;
  mode: Mode;
}

export interface ScaleData {
  root: string;
  notes: number[];
  octaveMin: number;
  octaveMax: number;
}

export type ChordType =
  | "Maj"
  | "Min"
  | "Dim"
  | "Aug"
  | "Sus2"
  | "Sus4"
  | "7"
  | "Maj7"
  | "Min7";

export interface ChordData {
  root: string;
  type: ChordType;
  inversion: number;
  notes: number[];
}

export interface ChordProgressionData {
  chords: ChordData[];
  barsPerChord: number;
}
```

調、スケール、コードは互換のない独立した型として扱い、文字列のみの暗黙変換は行いません。

## Musical Event

作曲ノードで使用する基本単位をNoteDataとNoteSequenceとして統一します。

```typescript
export interface NoteData {
  pitch: number;
  velocity: number;
  startBeat: number;
  duration: number;
  channel?: number;
}

export interface NoteSequence {
  notes: NoteData[];
  lengthBeats: number;
}
```

- `startBeat`と`duration`は小節位置ではなく拍単位
- `pitch`はMIDIノート番号
- `velocity`は0～127
- `channel`は楽器またはトラック固有の値

Compositionノードは入力と出力を可能な限りNoteDataおよびNoteSequenceへ集約します。

## Composition

### Motif

```typescript
export interface MotifData {
  notes: NoteData[];
  bars: number;
  complexity: number;
}
```

### Phrase

```typescript
export interface PhraseData {
  notes: NoteData[];
  bars: number;
  motifCount: number;
}
```

### Harmony

```typescript
export interface HarmonyVoice {
  notes: NoteData[];
}

export interface HarmonyData {
  voices: HarmonyVoice[];
}
```

### Rhythm

```typescript
export interface RhythmEvent {
  beat: number;
  length: number;
  velocity: number;
}

export interface RhythmData {
  events: RhythmEvent[];
  density: number;
}
```

RhythmDataにはピッチを持たせません。リズムと音程を分離することで、Drum Generator、Ostinato、Melodyなどで同じリズムを再利用できます。

## Arrangement

ゲーム音楽では、Sectionだけでなく、楽器レイヤーの有効状態を管理します。

```typescript
export interface ArrangementLayer {
  name: string;
  enabled: boolean;
  weight: number;
}

export interface ArrangementData {
  layers: ArrangementLayer[];
}
```

一般的なレイヤー例：

```text
Drums
Bass
Pad
Strings
Choir
```

`weight`は0～1に正規化し、各レイヤーの音量だけでなく、音符密度や有効化条件へ利用できます。

## Adaptive Music

```typescript
export interface MoodData {
  name: string;
  intensity: number;
}

export interface EnergyData {
  value: number;
}

export interface TransitionData {
  sourceSection: string;
  targetSection: string;
  durationBars: number;
}
```

MusicIntentやゲームパラメータからMoodDataとEnergyDataを作り、CompositionおよびArrangementノードへ接続します。

## MIDI

```typescript
export interface MidiTrack {
  name: string;
  notes: NoteData[];
}

export interface MidiData {
  ppq: number;
  tracks: MidiTrack[];
}
```

NoteSequenceは演奏単位、MidiDataはファイル出力単位です。トラック名、チャンネル、PPQなどを含むMIDI固有の情報はMidiData側で保持します。

## Audio

```typescript
export interface AudioData {
  sampleRate: number;
  channels: number;
  frameCount: number;
  samples: Float32Array[];
}
```

UIへ音声全体を複製せず、Rust Coreとの境界では次を検討します。

- バッファIDで管理する
- `SharedArrayBuffer`へ共有する
- Web WorkerやWASMへメモリを移動する
- Tauriではオーディオ処理をRust側に保持する

SharedArrayBufferを使う場合は、Cross-Origin Isolationが必要な実行環境とWebAssembly対応を別途確認します。

## Parameters

ゲーム連携の入力を型付きで表現します。

```typescript
export interface ParameterData<T = number> {
  name: string;
  value: T;
}

export type DangerParameter = ParameterData<number>;
export type BattleParameter = ParameterData<boolean>;
```

ただし、Compositionノードはゲーム固有のパラメータ名に依存せず、EnergyDataやMoodDataへ変換して受け取る構成を基本とします。

```text
DangerParameter
  ↓
Parameter Mapping
  ↓
EnergyData
  ↓
Adaptive Melody
```

## Trigger

```typescript
export interface TriggerData {
  id: UUID;
  fired: boolean;
  timestamp: number;
}
```

TriggerDataはSection切り替え、音符生成開始、楽曲終了などの瞬間イベントに使用します。

## Node Port

ReactFlowで描画するポートには、型と複数接続の可否を定義します。

```typescript
export interface PortDefinition {
  id: string;
  name: string;
  dataType: PortType;
  multiple?: boolean;
}
```

接続時の基本規則：

1. 出力`dataType`と入力`dataType`が一致する
2. `multiple`がfalseの場合、既存接続を置き換えるか拒否する
3. NoteSequenceとAudioDataを暗黙変換しない
4. `multiple`入力には同一型だけを接続する

## Base Node

```typescript
export interface KNode<TSettings = unknown> {
  id: UUID;
  type: string;
  name: string;
  version: string;
  settings: TSettings;
  inputs: PortDefinition[];
  outputs: PortDefinition[];
}
```

`type`は実装ノードの識別子、`version`は設定やポート仕様の互換性管理に利用します。

例：

```typescript
interface MotifSettings {
  lengthBars: 2 | 4 | 8;
  complexity: number;
  direction: "ascending" | "descending" | "mixed";
}

type MotifNode = KNode<MotifSettings>;
```

## Workflow

```typescript
export interface NodeConnection {
  sourceNodeId: UUID;
  sourcePortId: string;
  targetNodeId: UUID;
  targetPortId: string;
}

export interface KMusicGraph {
  id: UUID;
  version: string;
  nodes: KNode[];
  connections: NodeConnection[];
}
```

Workflowではノード定義と接続だけを保存し、音声バッファや大規模MIDI配列は必要に応じて外部アセット参照とします。

```json
{
  "id": "workflow-id",
  "version": "1.0.0",
  "nodes": [],
  "connections": []
}
```

## AI Composer

AIが生成する型は、完成した音符ではなく作曲指示です。

```typescript
export interface MusicIntent {
  genre: string;
  mood: string;
  bpm: number;
  complexity: number;
  energy: number;
  instruments: string[];
  progressionStyle: string;
  motifStyle: string;
}
```

例：

```json
{
  "genre": "JRPG",
  "mood": "Town",
  "bpm": 92,
  "complexity": 45,
  "energy": 25,
  "instruments": ["Shamisen", "Koto", "Flute"],
  "progressionStyle": "traditional",
  "motifStyle": "ascending"
}
```

処理経路：

```text
MusicIntent
  ↓
Motif
  ↓
Phrase
  ↓
Variation
  ↓
Counter Melody
  ↓
Harmony
  ↓
Arrangement
  ↓
Performance
```

AI出力はRuntime Validationで検証し、許可された値・範囲以外をRust Composition Engineへ渡しません。

## Rust Coreとの共有方針

`@kmusic/types`を実装の単一の型情報として扱いつつ、Rustとの共有にはシリアライズ可能な型だけを境界へ置きます。

```text
TypeScript Domain Model
  ↓ JSON / MessagePack
Rust Command Model
  ↓
Composition / MIDI / DSP Engine
```

| TypeScript型 | Rust側の扱い |
|-------------|-------------|
| NoteData / NoteSequence | 構造体としてデシリアライズ |
| ChordData / PhraseData | serde互換構造体 |
| MidiData | MIDI Engine入力 |
| AudioData.samples | 共有メモリまたはバッファID |
| KNode.settings | JSON Valueからノード固有型へ変換 |
| MusicIntent | Rustへ渡す前に検証 |

実際の同期には、TypeScript型とRust構造体から生成された共通スキーマを利用し、フィールド追加時のずれを検出します。

## 関連ドキュメント

- [MusicGraph](../index.md)
- [ノードベースのBGM作成](./ノードベースのBGM作成.md)
- [Composition層のノード構成](./Composition層のノード構成.md)
- [ノードごとの入出力型](./ノードごとの入出力型.md)
