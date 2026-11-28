# IP-Adapter

## 目次

- [概要](#概要)
- [基本処理フロー](#基本処理フロー)
- [入出力](#入出力)
- [主要パラメーター](#主要パラメーター)
- [IP-Adapter のWeight（重み）](#ip-adapter-のweight重み)
- [IP-Adapterの重みの範囲](#ip-adapterの重みの範囲)
- [重みを正規化する式](#重みを正規化する式)
- [複数の参照画像を使用する場合](#複数の参照画像を使用する場合)
- [複数画像の合成処理](#複数画像の合成処理)
- [部位別参照](#部位別参照)
- [袖用マスクの作り方](#袖用マスクの作り方)
- [袖マスクを自動抽出するコード](#袖マスクを自動抽出するコード)
- [DWPose のskeleton.json形式に合わせて修正](#dwpose-のskeletonjson形式に合わせて修正)
- [部位別マスク合成の実装を設計](#部位別マスク合成の実装を設計)
- [スプライト生成での利用](#スプライト生成での利用)
- [注意点](#注意点)
- [KImageGraphでのノード構成](#kimagegraphでのノード構成)
- [関連ドキュメント](#関連ドキュメント)

---

## 概要

IP-Adapterは、参照画像の視覚的特徴を画像生成モデルへ渡すためのアダプターです。テキストプロンプトだけでは固定しにくい、キャラクターの顔、髪型、服装、配色、画風などを参照画像から生成結果へ反映できます。

KImageGraphでは、参照画像をIP-Adapterへ入力し、ControlNetで構図やポーズを制御し、SDXLなどの画像生成モデルへ送る構成で利用します。

IP-Adapterはポーズ制御用ではありません。ポーズや関節位置はDWPose / OpenPose、輪郭や構図はControlNetなどで別に制御し、役割を分離します。

## 基本処理フロー

```text
Reference Image
    ↓
Image Encoder
    ↓
IP-Adapter
    ↓
Text Prompt + Model Features
    ↓
SDXL
```

ポーズ制御を組み合わせる場合は、次のように処理を統合します。

```text
Reference Image ──→ IP-Adapter
                         ↓
Text Prompt ─────────→ SDXL ──→ Output PNG
                         ↑
DWPose PNG ─────→ ControlNet
```

## 入出力

### 入力

- 参照画像
- テキストプロンプト
- IP-Adapterの重み
- 対応する画像生成モデル
- 必要に応じたControlNet画像

### 出力

- 参照画像の特徴を反映した生成画像
- ゲーム用途では、個別スプライトまたはスプライトシート用画像

## 主要パラメーター

| パラメーター | 役割 | 設定の目安 |
|-------------|------|-----------|
| Weight | 参照画像の影響度 | `0.5`～`1.0` |
| Reference Image | 反映する見た目の基準画像 | キャラクターが明瞭な画像 |
| Prompt Strength | テキストによる指定の強さ | モデル設定に依存 |
| Control Strength | ポーズや構図の反映強度 | ポーズを優先する場合は高め |
| Start / End Step | IP-Adapterを適用する生成期間 | 全期間または中盤まで |

設定値は使用するIP-Adapterモデル、ベースモデル、ControlNetの種類によって調整が必要です。

## IP-Adapter のWeight（重み）

IP-Adapterの参照画像が生成結果へ与える影響度は、一般に**Weight（重み）**と呼びます。
ComfyUIやIP-Adapter系ノードでは、`weight`または`ip_adapter_weight`という名前が使われることが多いです。

```text
Reference Image
    ↓
IP-Adapter
    ↓
SDXL
```

```json
{
  "weight": 0.6
}
```

重みの大まかな意味は、次のとおりです。

```text
0.0 = 参照画像を無視
0.5 = 少し似せる
1.0 = 強く似せる
1.5 = かなり強く似せる
```

値の範囲と効果は使用するIP-Adapter実装に依存します。高すぎる値では参照画像の見た目だけでなく構図やポーズまで強く反映され、ControlNetとの役割が競合する場合があります。

### 複数参照画像の指定

KImageGraphでは、参照画像ごとに重みを保持する形式をおすすめします。

```json
{
  "references": [
    {
      "image": "face.png",
      "weight": 0.8
    },
    {
      "image": "clothes.png",
      "weight": 1.2
    }
  ]
}
```

### 2種類の重み

複数参照画像を扱う場合は、次の2種類を分けて設計します。

#### Adapter Weight

IP-Adapter全体の効果を調整します。

```json
{
  "adapter_weight": 0.8
}
```

#### Image Weight

複数の参照画像間で優先度を調整します。

```json
{
  "face_weight": 0.3,
  "clothes_weight": 0.5,
  "weapon_weight": 0.2
}
```

KImageGraphではImage Weightを**Reference Weight**と呼び、参照画像の役割と重みを同じオブジェクトで表現すると分かりやすくなります。

```json
{
  "type": "ReferenceImage",
  "weight": 1.0,
  "role": "clothes"
}
```

Direction、Face、Clothes、Weaponを参照画像として分ける場合は、次の役割名を使用できます。

```text
Direction
Face
Clothes
Weapon
```

Adapter WeightでIP-Adapter全体の影響度を決め、Reference Weightで役割ごとの参照画像の優先度を決める構成にすると、設定の意味を混同せず管理できます。

## IP-Adapterの重みの範囲

KImageGraphの標準UIでは、IP-Adapter Weightの範囲を`0.0～2.0`、初期値を`1.0`に固定することを推奨します。

### 推奨仕様

```json
{
  "weight": 1.0
}
```

```json
{
  "minimum": 0.0,
  "maximum": 2.0,
  "default": 1.0
}
```

### 重みごとの意味

| Weight | 影響 | 想定用途 |
|-------:|------|----------|
| 0.00 | IP-Adapter無効 | 参照画像を使用しない |
| 0.25 | わずかに影響 | 髪色や雰囲気だけ寄せる |
| 0.50 | 弱い | 顔や服装は変化する可能性がある |
| 1.00 | 標準 | 通常使用時の推奨デフォルト |
| 1.25 | やや強い | キャラクターの一貫性を高める |
| 1.50 | 強い | 顔や服装を維持する |
| 2.00 | かなり強い | 参照画像へ強く寄せる |

`2.0`付近ではControlNetのポーズと参照画像のポーズが競合する場合があるため、スプライト生成では生成結果をフレーム単位で確認します。

### UI

0.01刻みのスライダーと数値入力を併用します。

```text
┌────────────────────┐
│ Weight : 1.00      │
├────────────────────┤
│ 0.00 ───●─── 2.00  │
└────────────────────┘
```

### 用途別デフォルト

#### Direction参照

```json
{
  "weight": 0.8
}
```

キャラクターの向きを参照しつつ、Pose画像の制御を残します。

#### Face参照

```json
{
  "weight": 1.2
}
```

小さいスプライトでも顔の特徴を維持するため、標準よりやや強めに設定します。

#### Clothes参照

```json
{
  "weight": 1.5
}
```

長袖・半袖の変化や、衣装の細部を維持する用途で使用します。

#### Weapon参照

```json
{
  "weight": 1.3
}
```

武器の種類、形状、相対的なサイズを維持します。

### Reference Pack

Direction、Face、Clothes、Weaponをまとめて管理する場合は、役割と重みを個別に指定します。

```json
{
  "references": [
    {
      "role": "direction",
      "weight": 0.8
    },
    {
      "role": "face",
      "weight": 1.2
    },
    {
      "role": "clothes",
      "weight": 1.5
    },
    {
      "role": "weapon",
      "weight": 1.3
    }
  ]
}
```

### 袖の一貫性を最優先するモード

```json
{
  "direction": 0.8,
  "face": 1.0,
  "clothes": 1.8,
  "weapon": 0.6
}
```

Clothes参照を他の参照より強くし、服装の一貫性を優先します。

### JSON Schema

```json
{
  "weight": {
    "type": "number",
    "minimum": 0.0,
    "maximum": 2.0,
    "default": 1.0
  }
}
```

内部APIでは将来の拡張に備えて`0.0～4.0`を許可し、標準UIでは安全な範囲として`0.0～2.0`だけを表示できます。

```text
内部許可範囲: 0.0 ～ 4.0
UI表示範囲:  0.0 ～ 2.0
通常利用範囲: 0.5 ～ 1.8
```

内部範囲とUI範囲を分離することで、高度なワークフローでは拡張値を扱いながら、一般ユーザー向けの設定崩れを防げます。

## IP-Adapter の weight

### Weight の正確な意味

厳密には、**Weight=1.2 → 参照画像に120%似る** という意味ではありません。

IP-Adapter の weight は**参照画像の特徴をどれだけ強く生成へ反映するか**という係数です。

### 体感的な目安

| Weight | 参照画像への追従 |
|--------|-----------------|
| 0.2 | 色や雰囲気だけ少し反映 |
| 0.5 | 髪色・服色が反映 |
| 0.8 | 顔や服の特徴がかなり反映 |
| 1.0 | 標準 |
| 1.2 | 顔・服・髪型の維持率が高い |
| 1.5 | 強く固定される |
| 2.0 | かなり参照画像寄り |

### 長袖キャラを参照した場合の例

例えば、長袖キャラクターを参照した場合の weight 別の変化を見てみます。

#### weight = 0.5

- 長袖→半袖になることがある
- 髪型も少し変わる

#### weight = 1.0

- 長袖維持率が高い
- 髪型も概ね維持

#### weight = 1.2

- 長袖維持率がさらに高い
- 顔の特徴も維持しやすい

#### weight = 1.8

- ほぼ長袖固定
- 代わりに**ポーズ変化**や**表現力**が少し失われる

### Character LoRA + DWPose + IP-Adapter 構成での推奨値

あなたの Character LoRA + DWPose + IP-Adapter 構成なら、次のように設定することをおすすめします。

#### 一般用途

```json
{
  "weight": 1.0
}
```

#### 顔の一貫性重視

```json
{
  "weight": 1.2
}
```

#### 袖・服装の一貫性重視

```json
{
  "weight": 1.4
}
```

#### 完全固定モード

```json
{
  "weight": 1.8
}
```

特にスプライト生成では、次のように参照の種類ごとに weight を分ける設定が有効です。

```json
{
  "face_reference": 1.0,
  "clothes_reference": 1.4,
  "weapon_reference": 0.7
}
```

**袖の一貫性を最優先にしたいなら、服画像だけ高め(1.3～1.5) に設定するのがバランスが良いです。**

### 上限は 2.0 なのか？

いいえ、**2.0 は実装上の上限ではありません。**

実際の IP-Adapter では、多くの実装で `0.0～3.0` や `0.0～10.0` のような値も入力できます。

ただし、実用上は次のような目安になります。

| Weight | 状態 |
|-------:|------|
| 0.0 | 無効 |
| 0.3～0.7 | 弱い参照 |
| 0.8～1.2 | 標準 |
| 1.2～1.5 | 強い参照 |
| 1.5～2.0 | 非常に強い |
| 2.0以上 | 過剰になりやすい |

例えば `face.png` に `weight=3.0` とすると：

- 顔はかなり維持される
- 反面、**ポーズへの追従**、**服の自然な変形**、**表情変化**が弱くなることがある

### Character LoRA + DWPose + IP-Adapter の制約

スプライト生成用途では特に、**IP-Adapter + Character LoRA + DWPose の3つが同時に効くため、weight を上げすぎると制約同士が競合します。**

#### UI 設計の推奨案

KImageGraph の UI なら、次のような設計が良いでしょう。

```json
{
  "minimum": 0.0,
  "maximum": 2.0,
  "default": 1.0
}
```

を公開範囲にして、内部的には以下を許可します。

```json
{
  "minimum": 0.0,
  "maximum": 4.0
}
```

つまり：

- `0.0～2.0` を通常モード
- `2.0～4.0` を「上級者モード」

にするイメージです。

### 最終的な推奨値

長袖・髪型・装備の維持が目的なら、経験的には次の値が最もバランスが良く、`2.0` を超えるケースはそれほど多くありません。

```json
{
  "face": 1.0,
  "clothes": 1.3,
  "weapon": 0.7
}
```

より細かく調整する場合は次の範囲を参考にしてください。

```json
{
  "face": 1.0,
  "clothes": 1.3～1.6,
  "weapon": 0.7～1.2
}
```

## 重みを正規化する式

複数画像へIP-Adapter Weightを配分する場合は、Reference Weightの総和を`1.0`へ正規化する方法をおすすめします。

### 基本式

全重みの合計を求めます。

```text
S = w1 + w2 + w3 + ... + wn
```

各重みを合計で割ります。

```text
w'i = wi / S
```

### 計算例

元の設定値です。

```json
{
  "direction": 0.8,
  "face": 1.2,
  "clothes": 1.8,
  "weapon": 0.6
}
```

合計は次のとおりです。

```text
S = 0.8 + 1.2 + 1.8 + 0.6
  = 4.4
```

各値を正規化します。

```text
direction = 0.8 / 4.4 = 0.182
face      = 1.2 / 4.4 = 0.273
clothes   = 1.8 / 4.4 = 0.409
weapon    = 0.6 / 4.4 = 0.136
```

正規化後の設定です。

```json
{
  "direction": 0.182,
  "face": 0.273,
  "clothes": 0.409,
  "weapon": 0.136
}
```

小数点3桁で丸めた場合も、合計を`1.000`に調整します。

```text
0.182
0.273
0.409
+ 0.136
───────
1.000
```

### KImageGraph向けTypeScript

```typescript
function normalizeWeights(
  weights: Record<string, number>,
): Record<string, number> {
  const total = Object.values(weights).reduce(
    (sum, value) => sum + value,
    0,
  );

  if (total <= 0) {
    return { ...weights };
  }

  const result: Record<string, number> = {};

  for (const [key, value] of Object.entries(weights)) {
    result[key] = value / total;
  }

  return result;
}
```

合計が`0`以下の場合に入力値をそのまま返すため、全画像が無効な設定でゼロ除算は発生しません。

使用例です。

```typescript
const weights = {
  direction: 0.8,
  face: 1.2,
  clothes: 1.8,
  weapon: 0.6,
};

const normalized = normalizeWeights(weights);
```

### 袖優先プリセット

衣装、特に長袖と半袖の一貫性を優先する入力値です。

```json
{
  "direction": 0.8,
  "face": 1.0,
  "clothes": 2.0,
  "weapon": 0.2
}
```

合計`4.0`で正規化した結果です。

```json
{
  "direction": 0.200,
  "face": 0.250,
  "clothes": 0.500,
  "weapon": 0.050
}
```

```text
服（袖）50%
顔      25%
方向    20%
武器     5%
```

Clothesへ最大の配分を与えることで、袖の形状や衣装の揺らぎを抑えやすくなります。

### ReferencePackBuilderで保持する値

ユーザー設定値と実際の適用値を確認できるよう、各参照画像に元の重みと正規化後の重みを保持します。

```json
{
  "image": "clothes.png",
  "weight": 2.0,
  "normalizedWeight": 0.500
}
```

- `weight`：ユーザーが設定した元の重み
- `normalizedWeight`：Reference Pack内の配分として使用する値

Adapter WeightはIP-Adapter全体の効き具合として別に保持し、Reference Weightの正規化値とは混在させません。

## 複数の参照画像を使用する場合

複数の参照画像を使う構成は、大きく4つの方式へ分けられます。

### 1. 単純加重平均方式

最も実装しやすい方式です。画像ごとのWeightを正規化し、特徴Embeddingを加重合成します。

```json
{
  "references": [
    {
      "role": "direction",
      "weight": 0.8
    },
    {
      "role": "face",
      "weight": 1.0
    },
    {
      "role": "clothes",
      "weight": 2.0
    }
  ]
}
```

```text
S = 0.8 + 1.0 + 2.0
  = 3.8
```

正規化後の値です。

```text
direction = 0.211
face      = 0.263
clothes   = 0.526
```

内部処理のイメージです。

```text
Direction Embedding × 0.211
Face Embedding      × 0.263
Clothes Embedding   × 0.526
            ↓
     合成Embedding
```

実装が単純で、Reference Pack内の画像を同じ形式でまとめて扱えるため、最初の実装方式としておすすめです。

### 2. 部位別IP-Adapter方式

画像の役割ごとにIP-Adapterを適用します。

```text
Direction Image → IP-Adapter
Face Image      → IP-Adapter
Clothes Image   → IP-Adapter
Weapon Image    → IP-Adapter
```

各参照画像の役割です。

```text
Direction → 向き
Face      → 顔
Clothes   → 衣装
Weapon    → 装備
```

部位ごとに影響を個別調整できるため、スプライト向けでは衣装や顔を固定しやすくなります。
ただし、IP-Adapterを複数実行するため、単純加重平均方式より処理負荷が高くなります。

長袖・半袖の変化を抑える場合は、Clothes参照の重みを最も強くします。

### 3. マルチモーダル埋め込み方式

各参照画像をCLIPなどのImage Encoderでベクトルへ変換します。

```text
Face Image
    ↓
Vector A

Clothes Image
    ↓
Vector B

Weapon Image
    ↓
Vector C
```

正規化した重みを使って、1つの埋め込みを生成します。

```text
FinalVector
    = A × wa
    + B × wb
    + C × wc
```

埋め込み段階で統合するため、画像役割を統一的に扱えます。
KImageGraphでは、Reference Packをエンコードして`FinalVector`をIP-Adapterへ渡す方式として拡張できます。

### 4. Reference Pack方式

キャラクターごとの参照画像、役割、重みを1つのアセットとして管理します。

```json
{
  "id": "hero_reference_pack",
  "references": [
    {
      "role": "direction",
      "image": "left_master.png",
      "weight": 0.8
    },
    {
      "role": "face",
      "image": "face_reference.png",
      "weight": 1.0
    },
    {
      "role": "clothes",
      "image": "clothes_reference.png",
      "weight": 2.0
    },
    {
      "role": "weapon",
      "image": "weapon_reference.png",
      "weight": 0.2
    }
  ]
}
```

推奨するノード構成です。

```text
Reference Pack
    ↓
Weight Normalizer
    ↓
Multi Reference Encoder
    ↓
IP-Adapter
```

Reference Packとして保存すると、キャラクターごとの参照画像と重み設定を再利用できます。

### スプライト生成向け推奨重み

通常は各役割を均等に配分します。

```json
{
  "direction": 1.0,
  "face": 1.0,
  "clothes": 1.0,
  "weapon": 1.0
}
```

```text
Direction 25%
Face      25%
Clothes   25%
Weapon    25%
```

長袖を固定するモードでは、Clothesの重みを強くします。

```json
{
  "direction": 0.8,
  "face": 1.0,
  "clothes": 2.0,
  "weapon": 0.2
}
```

合計`4.0`で正規化すると、次の配分です。

```text
direction = 20.0%
face      = 25.0%
clothes   = 50.0%
weapon    = 5.0%
```

顔、服、武器、向きを別々に管理することで、次の変化を抑えやすくなります。

```text
長袖 → 半袖
髪型の変化
武器の変化
```

### KImageGraphの標準構成

```text
ReferencePack
    ↓
WeightNormalizer
    ↓
MultiReferenceEncoder
    ↓
IP-Adapter
```

この構成にすると、参照画像の追加、重み調整、埋め込み生成を独立したノードとして扱えます。

## 複数画像の合成処理

KImageGraphでは、複数画像をピクセル単位で重ね合わせるのではなく、各画像をCLIP EncoderでEmbeddingへ変換してから重み付き合成します。

```text
Reference Images
    ↓
CLIP Encoder
    ↓
Embeddings
    ↓
Weighted Composition
    ↓
IP-Adapter
```

この方式では、画像本体ではなく、顔、髪、衣装、装備などの特徴ベクトルを統合します。

### 方式1：線形加重平均

次の4つの参照画像を使用する場合を考えます。

```text
left_master.png
face.png
clothes.png
weapon.png
```

各画像を個別にエンコードします。

```text
left_master.png → E1
face.png        → E2
clothes.png     → E3
weapon.png      → E4
```

正規化済みの重みを指定します。

```json
{
  "direction": 0.2,
  "face": 0.3,
  "clothes": 0.4,
  "weapon": 0.1
}
```

合成式です。

```text
Efinal
    = E1 × 0.2
    + E2 × 0.3
    + E3 × 0.4
    + E4 × 0.1
```

一般式です。

```text
Efinal = Σ(Ei × Wi)
```

重みの合計を`1.0`にすると、合成Embeddingの尺度を保ちやすくなります。
単一ベクトルとしてIP-Adapterへ渡せるため、実装が単純で基本方式として推奨できます。

### 方式2：役割別埋め込み

参照画像を画像単位だけでなく、役割単位のEmbeddingとして保持します。

```text
left_master.png → Edirection
face.png        → Eface
clothes.png     → Eclothes
weapon.png      → Eweapon
```

```json
{
  "direction": "Edirection",
  "face": "Eface",
  "clothes": "Eclothes",
  "weapon": "Eweapon"
}
```

生成時に目的へ合わせて役割ごとの効き方を変更できます。

```text
顔を維持したい
    → Efaceを強くする

衣装を維持したい
    → Eclothesを強くする
```

スプライトではフレームごとに崩れやすい部位が異なるため、役割別Embeddingとして保持すると調整しやすくなります。

### 方式3：正規化合成

ユーザー入力の重みです。

```json
{
  "direction": 0.8,
  "face": 1.0,
  "clothes": 2.0,
  "weapon": 0.2
}
```

合計値を求めます。

```text
S = 0.8 + 1.0 + 2.0 + 0.2
  = 4.0
```

正規化後の値です。

```text
direction = 0.8 / 4.0 = 0.20
face      = 1.0 / 4.0 = 0.25
clothes   = 2.0 / 4.0 = 0.50
weapon    = 0.2 / 4.0 = 0.05
```

正規化された重みでEmbeddingを合成します。

```text
Efinal
    = Edirection × 0.20
    + Eface × 0.25
    + Eclothes × 0.50
    + Eweapon × 0.05
```

`weight`を表示・保存値、`normalizedWeight`を実際の合成値として分けて管理します。

### 方式4：Body Part方式

顔、髪、衣装、武器などをBody Partごとのアセットとして管理します。

```json
{
  "face": "face.png",
  "hair": "hair.png",
  "clothes": "clothes.png",
  "weapon": "weapon.png"
}
```

各画像を独立したEmbeddingへエンコードします。

```text
face.png    → Eface
hair.png    → Ehair
clothes.png → Eclothes
weapon.png  → Eweapon
```

出力構造の例です。

```json
{
  "faceEmbedding": "...",
  "hairEmbedding": "...",
  "clothesEmbedding": "...",
  "weaponEmbedding": "..."
}
```

生成側では、Hair Weight、Face Weight、Clothes Weight、Weapon Weightを個別に制御できます。
KImageGraphでは、参照画像の交換や部位別調整がしやすいため、このBody Part方式を拡張性の高い構成としておすすめします。

### 袖の一貫性を最優先する合成

長袖を固定する場合の正規化済み重みです。

```json
{
  "direction": 0.15,
  "face": 0.20,
  "clothes": 0.60,
  "weapon": 0.05
}
```

合成式です。

```text
Efinal
    = 0.15 × Edirection
    + 0.20 × Eface
    + 0.60 × Eclothes
    + 0.05 × Eweapon
```

服のEmbeddingを最も強くすることで、次の揺らぎを抑えやすくなります。

```text
長袖 → 半袖
コート → ジャケット
服色の変化
```

### MultiReferenceEncoderの内部構成

```text
Reference Images
    ↓
CLIP Encoder / Image Encoder
    ↓
Embeddings
    ↓
Weight Normalizer
    ↓
Weighted Sum
    ↓
IP-Adapter
```

MultiReferenceEncoderが重みの正規化と加重合成を担当し、IP-Adapterへは合成済みEmbeddingを渡します。

## 部位別参照

部位別参照は、単に**キャラ全身画像 → IP-Adapter** ではなく、**顔・髪・袖・服・武器**ごとに参照画像とマスクを持たせる方法です。

### 方法1 マスク付き参照画像（おすすめ）

まずマスター画像を分解します。

```text
master.png
    ↓
face.png + face_mask.png
hair.png + hair_mask.png
clothes.png + clothes_mask.png
weapon.png + weapon_mask.png
```

**例：元画像から服だけを抽出**

```text
元画像              clothes.png         clothes_mask.png
┌─────────┐
│  顔     │        (服部分)        (服部分だけ白、
│  髪     │                         他は黒)
│         │
│ コート  │
│         │
│  剣     │
└─────────┘
```

#### PartReference Schema

```json
{
  "role": "clothes",
  "image": "clothes.png",
  "mask": "clothes_mask.png",
  "weight": 1.5
}
```

### 方法2 Embedding を部位別管理

各部位を別々にエンコードします。

```text
face.png
    ↓
Face Embedding

hair.png
    ↓
Hair Embedding

clothes.png
    ↓
Clothes Embedding

weapon.png
    ↓
Weapon Embedding
```

#### Embedding の保持形式

```json
{
  "face_embedding": "...",
  "hair_embedding": "...",
  "clothes_embedding": "...",
  "weapon_embedding": "..."
}
```

#### 重み付け例（長袖固定）

```json
{
  "face": 0.15,
  "hair": 0.20,
  "clothes": 0.55,
  "weapon": 0.10
}
```

合成の計算式：

```text
Final = Face×0.15 + Hair×0.20 + Clothes×0.55 + Weapon×0.10
```

### 方法3 ControlNet 併用（スプライト用途推奨）

スプライト生成用途では、ControlNet と組み合わせることが最も効果的です。

```text
DWPose
    ↓
姿勢固定

Clothes Reference
    ↓
衣装固定

Face Reference
    ↓
顔固定
```

**結果：**

```text
ポーズは変更可能（DWPose で制御）
服は維持（Clothes Reference で固定）
顔は維持（Face Reference で固定）
```

### KImageGraph 向けノード構成

```text
Master Image
    │
    ▼
Part Extractor
    ├ Face
    ├ Hair
    ├ Clothes
    └ Weapon
    │
    ▼
Reference Pack Builder
    │
    ▼
Multi Reference Encoder
    │
    ▼
IP-Adapter
```

### 袖の色が変わる場合の対策

特に有効な設定は、以下の通りです。

#### 基本的な衣装固定

```json
{
  "role": "clothes",
  "weight": 1.5
}
```

#### 左右の袖を独立部位として扱う

さらに細かく調整する場合は、左右の袖を分離します。

```json
{
  "role": "left_sleeve",
  "weight": 1.8
}
```

```json
{
  "role": "right_sleeve",
  "weight": 1.8
}
```

このように分離すると、以下のようなブレをかなり抑えられます。

```text
長袖→半袖
青→水色
赤→茶色
```

### KImageGraph の標準部位構成

最終的に次の **6部位を標準化**するのがおすすめです。

```text
Face
Hair
LeftSleeve
RightSleeve
Clothes
Weapon
```

**特に LeftSleeve / RightSleeve を独立部位として扱うのが効果的です。** 衣装の一貫性だけ見ると、袖の色や形状を左右で独立管理することで、生成時のブレが大幅に減少します。

## 袖用マスクの作り方

袖マスクを作る方法はいくつかありますが、スプライト生成用途なら **「DWPose + セグメンテーション」** が最も実用的です。

### 方法1 手動作成（最初におすすめ）

まず1枚だけマスター画像を作ります。

```text
front_master.png
```

画像編集ソフトで **左袖** と **右袖** を選択し、マスク化します。

```text
left_sleeve_mask.png
right_sleeve_mask.png
```

**白黒画像フォーマット**

```text
白 = 袖
黒 = それ以外
```

**例：**

```text
■■□□□□
■■□□□□
■■□□□□
```

### 方法2 DWPose ベース自動生成

骨格情報から腕ROI（Region of Interest）を生成します。

#### ステップ1 骨格取得

```text
left_shoulder
left_elbow
left_wrist

right_shoulder
right_elbow
right_wrist
```

#### ステップ2 腕 ROI 生成

```text
肩 → 肘 → 手首
```

を太いポリゴン化して `ArmMask` を生成します。

#### ステップ3 袖マスク計算

```text
SleeveMask = ArmMask ∩ ClothesMask
```

**結果：** `SleeveMask` が生成されます。

### 方法3 人体セグメンテーション（最もおすすめ）

最も自動化でき、精度が高い方法です。

#### ステップ1 マスター画像をセグメント

```text
front_master.png
        ↓
Body Segmenter実行
```

#### ステップ2 セグメンテーション出力

```text
Hair
Face
UpperClothes
LowerClothes
Arm
Leg
Hand
```

#### ステップ3 袖マスク抽出

```text
SleeveMask = ArmMask ∩ UpperClothesMask
```

### KImageGraph 向けノード構成

```text
Master Image
    │
    ▼
Body Segmenter
    │
    ├ Hair Mask
    ├ Face Mask
    ├ UpperClothes Mask
    ├ Arm Mask
    │
    ▼
Sleeve Mask Builder
    │
    ▼
Left Sleeve Mask
Right Sleeve Mask
```

### Sleeve Mask Builder のアルゴリズム

#### 左腕ポリゴン

```text
UpperArmPolygon = Shoulder → Elbow
```

#### 前腕ポリゴン

```text
ForeArmPolygon = Elbow → Wrist
```

#### 腕マスク結合

```text
ArmMask = UpperArmPolygon + ForeArmPolygon
```

#### 袖マスク交差

```text
SleeveMask = ArmMask ∩ UpperClothesMask
```

### 保存形式

袖マスクの推奨保存形式は以下の通りです。

```json
{
  "part": "left_sleeve",
  "texture": "left_sleeve.png",
  "mask": "left_sleeve_mask.png",
  "followBones": [
    "left_shoulder",
    "left_elbow",
    "left_wrist"
  ]
}
```

### スプライト用途での実装優先順位

最初は次の **4種類だけ作る** と効果が大きいです。

```text
Hair Mask
Left Sleeve Mask
Right Sleeve Mask
Weapon Mask
```

この4つだけでも、以下のような大部分の問題を補正できます。

```text
髪色変化
長袖→半袖
武器サイズ変化
```

#### 長袖の一貫性を最優先する場合

```text
SleeveMask = ArmMask ∩ UpperClothesMask
```

で抽出し、マスター画像の袖テクスチャを各フレームの腕骨格へワープして再配置する方式が最も安定です。

## 袖マスクを自動抽出するコード

袖マスクを自動抽出するなら、**DWPose骨格 + セグメンテーションマスク** を組み合わせるのが最も安定です。

以下は、`UpperClothesMask + DWPose Skeleton → LeftSleeveMask + RightSleeveMask` を生成するコード例です。

### 前提

以下のファイルが存在することを想定しています。

```text
master.png
upper_clothes_mask.png
skeleton.json
```

### Python実装

```python
import json
import cv2
import numpy as np


def make_arm_polygon(
    shoulder,
    elbow,
    wrist,
    width_upper=40,
    width_lower=35
):
    """
    腕をポリゴン化
    """

    def perp(v):
        return np.array([-v[1], v[0]])

    shoulder = np.array(shoulder, dtype=np.float32)
    elbow = np.array(elbow, dtype=np.float32)
    wrist = np.array(wrist, dtype=np.float32)

    v1 = elbow - shoulder
    v2 = wrist - elbow

    v1 = v1 / max(np.linalg.norm(v1), 1e-6)
    v2 = v2 / max(np.linalg.norm(v2), 1e-6)

    n1 = perp(v1)
    n2 = perp(v2)

    poly = np.array([
        shoulder + n1 * width_upper,
        shoulder - n1 * width_upper,

        elbow - n1 * width_upper,
        elbow - n2 * width_lower,

        wrist - n2 * width_lower,

        wrist + n2 * width_lower,

        elbow + n2 * width_lower,
        elbow + n1 * width_upper
    ])

    return poly.astype(np.int32)


def sleeve_mask_from_pose(
    clothes_mask_path,
    skeleton_json_path,
    left_output,
    right_output
):

    clothes_mask = cv2.imread(
        clothes_mask_path,
        cv2.IMREAD_GRAYSCALE
    )

    h, w = clothes_mask.shape

    with open(
        skeleton_json_path,
        "r",
        encoding="utf-8"
    ) as f:

        pose = json.load(f)

    body = pose["body"]

    left_arm = np.zeros(
        (h, w),
        dtype=np.uint8
    )

    right_arm = np.zeros(
        (h, w),
        dtype=np.uint8
    )

    # ----------------------
    # 左腕
    # ----------------------

    poly = make_arm_polygon(
        body["left_shoulder"],
        body["left_elbow"],
        body["left_wrist"]
    )

    cv2.fillPoly(
        left_arm,
        [poly],
        255
    )

    # ----------------------
    # 右腕
    # ----------------------

    poly = make_arm_polygon(
        body["right_shoulder"],
        body["right_elbow"],
        body["right_wrist"]
    )

    cv2.fillPoly(
        right_arm,
        [poly],
        255
    )

    # ----------------------
    # 袖
    # ----------------------

    left_sleeve = cv2.bitwise_and(
        clothes_mask,
        left_arm
    )

    right_sleeve = cv2.bitwise_and(
        clothes_mask,
        right_arm
    )

    kernel = np.ones(
        (5, 5),
        np.uint8
    )

    left_sleeve = cv2.morphologyEx(
        left_sleeve,
        cv2.MORPH_CLOSE,
        kernel
    )

    right_sleeve = cv2.morphologyEx(
        right_sleeve,
        cv2.MORPH_CLOSE,
        kernel
    )

    cv2.imwrite(
        left_output,
        left_sleeve
    )

    cv2.imwrite(
        right_output,
        right_sleeve
    )


# --------------------------------
# 使用例
# --------------------------------

sleeve_mask_from_pose(
    "upper_clothes_mask.png",
    "skeleton.json",
    "left_sleeve_mask.png",
    "right_sleeve_mask.png"
)
```

### コード解説

#### make_arm_polygon()

```
shoulder → elbow → wrist
```

を太いポリゴンとして表現します。

- **v1, v2**: 肩→肘、肘→手首のベクトル
- **n1, n2**: 各ベクトルに垂直な単位ベクトル
- **width_upper, width_lower**: 上腕・前腕の幅（ピクセル）
- **poly**: 8頂点のポリゴン

左右で異なる幅を指定できるため、袖の形状に合わせて調整可能です。

#### sleeve_mask_from_pose()

1. **clothes_mask を読み込む**（グレースケール）
2. **skeleton.json から骨格座標を取得**
3. **左腕・右腕のポリゴンを生成**
4. **clothes_mask と腕マスクの交差を計算** → 袖マスク
5. **形態学的操作（MORPH_CLOSE）で穴埋め**
6. **結果を PNG で保存**

### 出力

```text
left_sleeve_mask.png
right_sleeve_mask.png
```

**フォーマット：**

- **白（255）**: 袖
- **黒（0）**: それ以外

### KImageGraph 向けの発展形

最終的には以下のようなノード構成にすると、複数部位を統一的に管理できます。

```text
Master Image
    ↓
Body Segmenter
    ↓
UpperClothes Mask
    +
DWPose Skeleton
    ↓
Sleeve Mask Builder
    ↓
LeftSleeve Asset
RightSleeve Asset
```

このアプローチにより、以下のように複数部品を部品化できます。

```text
髪
左袖
右袖
武器
マント
肩当て
```

すべて同じ仕組みで管理できるため、スケーラビリティが高くなります。

**特に長袖キャラのスプライト生成では、「ArmMask ∩ UpperClothesMask」の考え方が最も安定して袖領域を抽出できます。**

## DWPose のskeleton.json形式に合わせて修正

前回のコードは以下のような KImageGraph 独自形式を前提にしていました。

```json
{
  "body": {
    "left_shoulder": [440, 280]
  }
}
```

実際の DWPose/OpenPose 形式に合わせるなら、一般的には以下のような構造から読み取る方が良いです。

```json
{
  "people": [
    {
      "pose_keypoints_2d": [
        x, y, score,
        x, y, score,
        ...
      ]
    }
  ]
}
```

### DWPose Body 18点対応

DWPoseの18点キーポイント索引は以下の通りです。

```python
BODY_POINTS = {
    "nose": 0,

    "neck": 1,

    "right_shoulder": 2,
    "right_elbow": 3,
    "right_wrist": 4,

    "left_shoulder": 5,
    "left_elbow": 6,
    "left_wrist": 7,

    "right_hip": 8,
    "right_knee": 9,
    "right_ankle": 10,

    "left_hip": 11,
    "left_knee": 12,
    "left_ankle": 13,

    "right_eye": 14,
    "left_eye": 15,

    "right_ear": 16,
    "left_ear": 17
}
```

### pose_keypoints_2d から座標取得

各キーポイントは `x, y, score` の3値で表現されます。

```python
def get_body_point(person, name):

    idx = BODY_POINTS[name]

    keypoints = person["pose_keypoints_2d"]

    x = keypoints[idx * 3]
    y = keypoints[idx * 3 + 1]
    score = keypoints[idx * 3 + 2]

    return (x, y, score)
```

### 袖マスク抽出用コード（DWPose JSON対応）

これなら DWPose/OpenPose JSON を直接使えます。

```python
import cv2
import json
import numpy as np


BODY_POINTS = {
    "right_shoulder": 2,
    "right_elbow": 3,
    "right_wrist": 4,

    "left_shoulder": 5,
    "left_elbow": 6,
    "left_wrist": 7
}


def get_point(person, point_name):

    idx = BODY_POINTS[point_name]

    keypoints = person["pose_keypoints_2d"]

    x = keypoints[idx * 3]
    y = keypoints[idx * 3 + 1]
    score = keypoints[idx * 3 + 2]

    return np.array([x, y], dtype=np.float32)


def build_arm_mask(
        shape,
        shoulder,
        elbow,
        wrist,
        upper_width=40,
        lower_width=35):

    mask = np.zeros(shape, dtype=np.uint8)

    points = np.array([
        shoulder,
        elbow,
        wrist
    ], dtype=np.int32)

    cv2.polylines(
        mask,
        [points],
        False,
        255,
        upper_width
    )

    cv2.circle(
        mask,
        tuple(elbow.astype(int)),
        upper_width,
        255,
        -1
    )

    return mask


def extract_sleeve_masks(
        clothes_mask_path,
        dwpose_json_path,
        output_left,
        output_right):

    clothes_mask = cv2.imread(
        clothes_mask_path,
        cv2.IMREAD_GRAYSCALE
    )

    with open(
        dwpose_json_path,
        "r",
        encoding="utf-8"
    ) as f:

        data = json.load(f)

    person = data["people"][0]

    left_arm_mask = build_arm_mask(
        clothes_mask.shape,

        get_point(person, "left_shoulder"),
        get_point(person, "left_elbow"),
        get_point(person, "left_wrist")
    )

    right_arm_mask = build_arm_mask(
        clothes_mask.shape,

        get_point(person, "right_shoulder"),
        get_point(person, "right_elbow"),
        get_point(person, "right_wrist")
    )

    left_sleeve = cv2.bitwise_and(
        clothes_mask,
        left_arm_mask
    )

    right_sleeve = cv2.bitwise_and(
        clothes_mask,
        right_arm_mask
    )

    cv2.imwrite(
        output_left,
        left_sleeve
    )

    cv2.imwrite(
        output_right,
        right_sleeve
    )
```

### 使用方法

```python
extract_sleeve_masks(
    "upper_clothes_mask.png",
    "dwpose_skeleton.json",
    "left_sleeve_mask.png",
    "right_sleeve_mask.png"
)
```

### KImageGraph 拡張形式対応

もしあなたの KImageGraph の Skeleton JSON が以前設計した形式の場合：

```json
{
  "body": {},
  "face": {},
  "left_hand": {},
  "right_hand": {},
  "left_foot": {},
  "right_foot": {}
}
```

むしろそちらの方が処理しやすいです。

#### 将来的な推奨設計

将来的には以下のように拡張して、袖マスクを Skeleton Asset の一部として保持する設計がおすすめです。

```json
{
  "body": {},
  "face": {},
  "left_hand": {},
  "right_hand": {},
  "left_foot": {},
  "right_foot": {},

  "masks": {
    "hair": "...",
    "left_sleeve": "...",
    "right_sleeve": "...",
    "weapon": "..."
  }
}
```

こうすると、フレームごとの袖補正や Part Composer との連携がかなり楽になります。

## 部位別マスク合成の実装を設計

AI生成後に髪、袖、衣装、武器などの見た目が崩れた場合は、保存済みの部位画像を生成フレームへ後合成して補正できます。
KImageGraphでは、部位ごとのテクスチャとマスクをまとめたPart Asset方式を推奨します。

### 全体構成

```text
Master Image
    ↓
Part Extractor
    ├─ Face Mask
    ├─ Hair Mask
    ├─ Sleeve Mask
    ├─ Clothes Mask
    └─ Weapon Mask
    ↓
Part Asset Pack

Generated Frame
    ↓
Skeleton Detector / DWPose Skeleton
    ↓
Part Composer
    ↓
Corrected Frame
```

Part Extractorは基準画像から再利用する部位を作成し、PartComposerは生成フレームの骨格へ部位を追従させます。

### Part Asset

長袖キャラクターのPart Asset Pack例です。

```json
{
  "parts": {
    "hair": {
      "texture": "hair.png",
      "mask": "hair_mask.png"
    },
    "left_sleeve": {
      "texture": "left_sleeve.png",
      "mask": "left_sleeve_mask.png"
    },
    "right_sleeve": {
      "texture": "right_sleeve.png",
      "mask": "right_sleeve_mask.png"
    },
    "weapon": {
      "texture": "sword.png",
      "mask": "sword_mask.png"
    }
  }
}
```

テクスチャとマスクを常に対として保存し、差し替え時に座標系が一致するよう管理します。

### Part Extractor

基準画像から再利用対象の部位を抽出します。

```text
front_master.png
    ↓
Part Extractor
    ├─ hair.png / hair_mask.png
    ├─ face.png / face_mask.png
    ├─ left_sleeve.png / left_sleeve_mask.png
    ├─ right_sleeve.png / right_sleeve_mask.png
    └─ weapon.png / weapon_mask.png
```

マスクはアルファ合成に使用し、テクスチャ外の背景を合成しないようにします。

### DWPose連携

現在のフレームから、パーツ追従に必要な関節を取得します。

```json
{
  "left_arm": {
    "shoulder": [400, 200],
    "elbow": [360, 260],
    "wrist": [320, 310]
  }
}
```

袖パーツでは、肩・肘・手首を制御点として使用します。

### ボーン追従ワープ

袖を肩から肘、肘から手首へ分割して変形します。

```text
UpperArmLength = |Shoulder - Elbow|
ForeArmLength  = |Elbow - Wrist|
```

基準ポーズとの倍率です。

```text
scale = CurrentLength / ReferenceLength
```

現在のボーン角度です。

```text
angle = atan2(dy, dx)
```

変換後の袖パーツ位置は、次の要素から求めます。

```text
PartTransform
    = Translation
    × Rotation
    × Scale
```

腕が大きく曲がる場合は画像全体を一括変形せず、上腕と前腕の領域へ分割し、`piecewise_affine`またはメッシュワープを使用します。

### 合成優先度

奥側から手前側へ描画します。

```text
0  Base Character
1  Clothes
2  Sleeves
3  Hair
4  Weapon
5  FX
```

```text
Base
    ↓
Clothes
    ↓
Sleeves
    ↓
Hair
    ↓
Weapon
    ↓
FX
```

背景ではなく生成フレーム自体をBaseとして使用するため、崩れた部位だけを上書きできます。

### アルファ合成

```text
OutColor
    = SrcColor × SrcAlpha
    + DstColor × (1 - SrcAlpha)
```

RGB値をマスクのアルファ値で補間し、境界を滑らかに合成します。
必要に応じてマスクへフェザー処理を加え、生成画像との境目を目立たせません。

### JSON Schema

```json
{
  "part": "left_sleeve",
  "zOrder": 2,
  "followBone": [
    "left_shoulder",
    "left_elbow",
    "left_wrist"
  ],
  "warpMode": "piecewise_affine"
}
```

`followBone`は追従する関節の順序を表し、`zOrder`は合成優先度を表します。

### PartComposer

入力例です。

```json
{
  "frame": "walk_03.png",
  "skeleton": "walk_03_pose.json",
  "part_assets": "hero_part_pack.json"
}
```

出力です。

```text
walk_03_fixed.png
```

PartComposerは、Part Asset Packを読み込み、骨格へ追従する変換を計算し、合成優先度に従ってマスク付き画像を重ねます。

### KImageGraph向けノード

```text
MasterImage
    ↓
PartExtractor
    ↓
PartAssetPack

GeneratedFrame
    ↓
SkeletonDetector
    ↓
BoneWarp
    ↓
PartComposer
    ↓
FrameCollection
```

PartAssetPackはキャラクターごとに保存し、異なるモーションフレームへ同じ部位データを再利用します。

### 最初に実装する部位

初期実装では、次の4部位から始めると効果を確認しやすくなります。

```text
Hair
LeftSleeve
RightSleeve
Weapon
```

その後、次の部位を追加します。

```text
Cape
ShoulderArmor
Skirt
Shield
```

生成画像で長袖が半袖へ崩れた場合でも、PartComposerで保存済みの袖テクスチャを現在の骨格へ追従させて再配置できるため、画像を再生成せず衣装の一貫性を補正できます。

## スプライト生成での利用

スプライトでは同一キャラクターを複数フレームで生成するため、参照画像とポーズ画像の両方を固定します。

```text
Character Reference
    ↓
IP-Adapter
    ↓
Pose PNG ──→ ControlNet
    ↓
SDXL
    ↓
Animation Frame
```

推奨する分担です。

- IP-Adapter：顔、髪、服、配色、画風
- ControlNet：向き、ポーズ、シルエット
- Prompt：背景、表情、動作の補足
- Seed：生成単位で固定または管理

歩行、攻撃、ジャンプなどの各フレームでは、参照画像と生成設定を共有し、変更するのは主にDWPose / OpenPose画像にします。

## 注意点

### 参照画像の選択

参照画像には、対象キャラクターが正面または用途に近い向きで明瞭に写っているものを使用します。

背景が複雑な画像を使用すると、キャラクター以外の背景要素も生成へ混入する場合があります。

### IP-Adapterの重み

重みが高すぎると、参照画像のポーズや構図まで再現し、ControlNetのポーズ制御が弱くなる場合があります。

```text
低い重み:
  テキストやポーズが優先

高い重み:
  参照画像の見た目が優先
```

### 単一参照の限界

単一画像だけでは、顔、衣装、装備などを個別に十分固定できない場合があります。複数の参照を使い分ける場合は、以下のガイドを参照してください。

- [Multi Reference IP-Adapter ガイド](Multi-Reference-IP-Adapter.md)

## KImageGraphでのノード構成

```json
{
  "reference_image": "character_master.png",
  "ip_adapter_weight": 0.8,
  "prompt": "full body game sprite, consistent character",
  "pose_image": "walk_03_pose.png"
}
```

ノード構成の例です。

```text
Character Reference
    ↓
IP-Adapter Node
    │
Text Prompt ─────┤
    │
Pose PNG → ControlNet Node
    │
    ▼
SDXL Generation
    ↓
Sprite PNG
```

バッチ処理では、同じIP-Adapter設定と参照画像を使用し、Pose PNGだけをフレームごとに切り替えます。

## 関連ドキュメント

- [Multi Reference IP-Adapter ガイド](Multi-Reference-IP-Adapter.md)
- [2頭身でポーズをつける](2頭身でポーズをつける.md)
- [2DSpriteGraph](../index.md)
