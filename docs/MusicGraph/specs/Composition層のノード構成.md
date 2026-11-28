# Composition層のノード構成

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

Composition層では、楽曲の構造と旋律を一つの「Melody Generator」に任せず、作曲工程を役割ごとのノードへ分割します。

現在の構成：

```text
Scale
  ↓
Melody Generator
  ↓
NoteSequence
```

この構成では、AIに作曲全体を任せるため、スケールに音を並べるだけの単調な結果になりやすくなります。

K-Music-Graphでは、次の工程へ分解します。

```text
楽曲構造
  ↓
モチーフ生成
  ↓
展開
  ↓
変奏
  ↓
対旋律
  ↓
和声・アレンジ
  ↓
演奏表現
  ↓
出力
```

## 理想的な構成

```text
Theory
  ↓
Motif
  ↓
Phrase
  ↓
Variation
  ├─ Main Melody
  └─ Counter Melody
       ↓
Harmony Layer
       ↓
Arrangement
       ↓
Performance
```

Composition層は、次の12ノードで構成します。

| ノード | 役割 |
|--------|------|
| Motif | 楽曲の核となる短いフレーズを生成する |
| Phrase | Motifを楽曲構造へ展開する |
| Variation | 移調、反行、逆行、リズム変化を加える |
| Sequence Generator | 短いパターンを段階的に反復・移動する |
| Counter Melody | 主旋律に対する対旋律を生成する |
| Harmony Generator | ChordからPad、Strings、Choirの和声を作る |
| Ostinato | 同じリズムやパターンを繰り返す |
| Ornament | 装飾音を追加する |
| Call & Response | フレーズと応答フレーズを作る |
| Transition | Section間を自然につなぐ |
| Density Control | フレーズの音数を制御する |
| Adaptive Melody | ゲームParameterに応じて旋律を変化させる |

## 共通データ型

Compositionノード間では、単なる音名配列ではなく、時間と演奏情報を含むデータを受け渡します。

```json
{
  "bars": 4,
  "timeSignature": "4/4",
  "notes": [
    {
      "pitch": 60,
      "start": 0.0,
      "duration": 1.0,
      "velocity": 82
    }
  ]
}
```

主なデータ型：

| 型 | 内容 |
|----|------|
| TheoryData | Key、Scale、Chord、BPM |
| MotifData | 短い旋律候補と作曲上の特徴 |
| PhraseData | 複数小節の旋律と構造情報 |
| HarmonySequence | 和声の声部配置 |
| NoteSequence | 最終的なMIDIノート列 |

## 1. Motif

楽曲の核となる4～8音程度の短いフレーズを生成します。

```text
Scale / Chord / Mood / Length
  ↓
Motif
  ↓
PhraseData
```

入力：

```text
ScaleData
Chord
```

パラメータ：

| パラメータ | 値 | 説明 |
|-----------|----|------|
| Length | 2 Bar / 4 Bar / 8 Bar | 生成する小節数 |
| Complexity | 0～100 | 音程、装飾、リズムの複雑さ |
| Direction | Ascending / Descending / Mixed | 旋律の進行方向 |

出力例：

```text
C D G E
```

Motifは、Phrase、Variation、Adaptive Melodyで共通して参照できる安定した楽曲素材です。

## 2. Phrase

Motifを展開し、曲の各Sectionで利用できるフレーズへ変換します。

```text
Motif
  ↓
Phrase
  ↓
PhraseData
```

Structureには、ゲームBGMで多用される反復構造を指定します。

```text
AB
ABAB
AABA
AABAC
```

例：

```text
A → A' → B → A
```

`A'`は完全な繰り返しではなく、終止位置や最後の数音だけを変えたAの展開を表します。

## 3. Variation

Phraseに移調、反行、逆行、リズム変化を加えます。

```text
Phrase
  ↓
Variation
  ↓
PhraseData
```

対応する変奏：

| Type | 説明 |
|------|------|
| Transpose | 指定音程だけ移調する |
| Invert | 音程を基準音に対して反転する |
| Retrograde | 音符の順序を逆にする |
| Rhythm | 音符のタイミングと長さを変化させる |
| Octave | 一部の音をオクターブ移動する |
| Random | 制約範囲内でランダムに変化させる |

例：

```text
元:         C D E G
Transpose:  D E F# A
Invert:     C Bb Ab F
Retrograde: G E D C
```

## 4. Sequence Generator

クラシックやゲームBGMで使われる、短いパターンの段階的な反復を生成します。

入力：

```text
PhraseData
```

パラメータ：

| パラメータ | 説明 |
|-----------|------|
| Step | 各反復で移動する音程 |
| Direction | 上行または下行 |
| Repeat | 反復回数 |

例：

```text
C D E
  ↓
D E F
  ↓
E F G
```

Sequenceはコード進行の範囲内に留めるか、次のコードへ解決するかを指定できるようにします。

## 5. Counter Melody

主旋律とコードを入力として、補完する対旋律を生成します。

```text
Main Melody + Chord
  ↓
Counter Melody
  ↓
CounterPhrase
```

Style：

| Style | 動作 |
|-------|------|
| Parallel | 主旋律と同じ方向へ進む |
| Contrary | 主旋律と反対方向へ進む |
| Free | コードを基準に独立した旋律を作る |

例：

```text
主旋律: C D E G
対旋律: G F E C
```

主旋律との重複や不協和を制御し、音域とリズムを分離して聞き分けられる声部にします。

## 6. Harmony Generator

コード進行から、Pad、Strings、Choirなどの和声レイヤーを生成します。

```text
Chord Progression
  ↓
Harmony Generator
  ↓
HarmonySequence
```

Style：

```text
Strings
Choir
Pad
Brass
```

例：

```text
C Maj7
  ↓
C - E - G - B
```

基本実装では、転回形、声部配置、音域、音の重複を制御します。

## 7. Ostinato

ゲームBGM向けに、同じリズムや音型をコードに応じて反復します。

```text
Chord
  ↓
Ostinato
  ↓
NoteSequence
```

例：

```text
C G E G
C G E G
C G E G
```

コードが変化してもリズムパターンを維持し、曲に推進力を与えます。Parameterから変奏量と密度を制御できます。

## 8. Ornament

Phraseの指定箇所へ装飾音を追加します。

```text
PhraseData
  ↓
Ornament
  ↓
PhraseData
```

対応する装飾：

| Type | 例 |
|------|----|
| Grace Note | D → Eへの導入に短いD#を加える |
| Trill | D E D Eという隣接音の反復 |
| Mordent | 主音と隣接音を短く往復する |
| Slide | 始点音から目標音へ連続的に移動する |

装飾量を過剰にしないよう、SectionとDensity Controlから上限を設定します。

## 9. Call & Response

一つのPhraseに対して、別のPhraseで応答する構造を生成します。JRPGの場面BGM、FluteとStringsの掛け合いなどに使用します。

```text
Phrase A
  ↓
Call & Response
  ↓
Phrase B
```

例：

```text
フルート: Call Phrase
          ↓
Strings: Response Phrase
```

応答Phraseでは、Motifの終止、音域、楽器、音数を変えながら、元のPhraseとの関連性を維持します。

## 10. Transition

TownからBattle、BattleからBossへ移動するときにBridge Phraseを生成します。

```text
Phrase A
  ↓
Transition
  ↓
Phrase B
```

入力：

```text
Phrase A
Phrase B
Target Section
Timing Rule
```

出力：

```text
BridgePhrase
```

切り替え方式：

- 次の小節頭で切り替える
- フィルを挿入する
- コードブリッジを生成する
- テンポや調を段階的に変更する
- レイヤーをクロスフェードする

Transitionは、音声素材の単純なフェードではなく、音楽的な拍とコードの接続を保証します。

## 11. Density Control

Phraseの音数、音域、同時発音数を制御します。

```text
Phrase
  ↓
Density Control: 0～100
  ↓
PhraseData
```

目安：

| Density | 結果 |
|---------|------|
| 0 | 最小限の旋律 |
| 20 | シンプルな音型 |
| 50 | 標準的な装飾と反復 |
| 80 | 多い音数と積極的な変奏 |
| 100 | 楽器・声部の上限まで使用 |

Densityを高くする場合も、キー、コード、リズム、楽器の演奏制約は維持します。

## 12. Adaptive Melody

K-Music-Graph独自のノードとして、Float Parameterを作曲構造へ接続します。

```text
Float Parameter: Danger
  ↓
Adaptive Melody
  ↓
Density / Leap / Ornament
```

例：

| Danger | 音数 | 跳躍 | 装飾 |
|--------|------|------|------|
| 0 | 20% | 10% | 0% |
| 25 | 35% | 20% | 10% |
| 50 | 60% | 45% | 30% |
| 100 | 90% | 80% | 60% |

入力Parameterに応じて、次のノードへ値を渡します。

```text
Danger
  ├─ Density Control
  ├─ Variation Amount
  ├─ Ornament Frequency
  ├─ Leap Limit
  └─ Transition Urgency
```

段階的な値変化では、拍やフレーズ境界に合わせて音楽状態を更新し、急な音符変更を避けます。

## AIノードとの接続

AIには完成したMIDIノート列を生成させず、作曲方針だけを返させます。

```text
Prompt
  ↓
AI Composer
  ↓
Composition Plan
  ↓
Rust Composition Engine
```

AI出力例：

```json
{
  "scale": "Japanese",
  "motif": "ascending",
  "phrase": "AABA",
  "variation": "transpose",
  "counter": true,
  "ornament": "grace_note"
}
```

構造化された値から、Rust Coreがノードごとの処理を実行します。

```text
Motif
  ↓
Phrase
  ↓
Variation
  ↓
Counter Melody
  ↓
Harmony / Ostinato / Ornament
  ↓
NoteSequence
```

AIは高レベルの選択を担当し、Rust Coreは音楽理論、制約、再現可能な音符生成を担当します。

## 最終構成

Composition層の完成形：

```text
Motif
  ↓
Phrase
  ↓
Variation
  ├─ Sequence Generator
  ├─ Counter Melody
  ├─ Harmony Generator
  └─ Ostinato
       ↓
Ornament
       ↓
Call & Response
       ↓
Density Control
       ↓
Adaptive Melody
       ↓
Transition
       ↓
NoteSequence
```

Adaptive Melody、Transition、Counter Melodyの3ノードは、K-Music-Graphを通常のDAWや単純な自動作曲ツールと差別化する中心機能です。

この構成により、次の工程をすべてノードグラフとして表現できます。

```text
ゲーム状況
  ↓
音楽理論
  ↓
作曲ロジック
  ↓
演奏
```

## 関連ドキュメント

- [MusicGraph](../index.md)
- [ノードベースのBGM作成](./ノードベースのBGM作成.md)
- [メロディーの改善](./メロディーの改善.md)
- [現実装の評価](./現実装の評価.md)
- [ノードごとの入出力型](./ノードごとの入出力型.md)
- [TypeScriptの型定義](./TypeScriptの型定義.md)
