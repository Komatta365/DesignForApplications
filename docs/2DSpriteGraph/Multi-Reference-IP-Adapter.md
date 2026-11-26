# Multi Reference IP-Adapter ガイド

## 結論

KImageGraph では、複数の参照画像を使用する以下のような構成が実現できます：

```
Plain Text + Direction Image + Face Image + Clothes Image + Weapon Image
```

例：
```
left_master.png
face_detail.png
clothes_detail.png
weapon_detail.png
```

---

## パターン1：単一参照（最もシンプル）

### 構成

```
left_master.png
    ↓
IP-Adapter
    ↓
SDXL
```

### 特徴

最もシンプルで処理も軽い。

### 問題点

以下の要素が変わることがあります：

- **向き** - 維持される
- **服** - 変わることがある
- **髪型** - 変わることがある
- **袖** - 変わることがある

---

## パターン2：複数IP-Adapter（段階的適用）

### 構成

```
left_master.png
    ↓
IP-Adapter
    ↓
face_detail.png
    ↓
IP-Adapter
    ↓
clothes_detail.png
    ↓
IP-Adapter
    ↓
weapon_detail.png
    ↓
IP-Adapter
    ↓
SDXL
```

### 特徴

複数の参照画像を段階的に適用します。実質的に複数画像を参照しているのと同じです。

### メリット

各IP-Adapterで細かな特徴を段階的に強化できます。

### デメリット

処理が重くなり、適用順序によって結果が変わる可能性があります。

---

## パターン3：Reference Pack（推奨）

### 構成

```json
{
  "direction": "left",
  "references": [
    "left_master.png",
    "face.png",
    "clothes.png",
    "weapon.png"
  ]
}
```

### 処理フロー

```
Reference Pack
    ↓
Reference Pack Builder
（Face、Clothes、Weapon を集める）
    ↓
Multi Reference IP Adapter
    ↓
SDXL
```

### KImageGraphへの実装イメージ

```
Character Master Pack
    │
    ├─ Front
    ├─ Left
    ├─ Right
    ├─ Back
    │
    ├─ Face
    ├─ Hair
    ├─ Clothes
    └─ Weapon
    │
    ▼
Reference Pack Builder
    │
    ▼
IP Adapter
    │
    ▼
SDXL
```

### メリット

- KImageGraph に最適な構成
- 複数画像を統一的に管理可能
- 実装がシンプル

---

## スプライト用途で重要な参照画像分割

キャラの一貫性を保つには、以下の要素を分けることが効果的です：

### 顔

```
face_reference.png
```

**抽出される特徴**：顔の形、目、鼻、口など

### 髪

```
hair_reference.png
```

**抽出される特徴**：髪型、髪色、髪の流れなど

### 服

```
clothes_reference.png
```

**抽出される特徴**：服の形、色、素材、パターンなど

### 装備

```
weapon_reference.png
```

**抽出される特徴**：武器の形、色、スタイルなど

---

## IP-Adapter の仕組み

IP-Adapter は **キャラクターの特徴を抽出する仕組み** です。

参照画像を増やすことで、以下の情報量が増えます：

```
参照画像 1枚  →  顔の特徴
参照画像 2枚  →  顔の特徴 + 髪型
参照画像 3枚  →  顔の特徴 + 髪型 + 服の特徴
参照画像 4枚  →  顔の特徴 + 髪型 + 服の特徴 + 武器の特徴
```

---

## スプライト生成向けのおすすめ構成

### 推奨セットアップ

```
Direction Master
    +
Face Reference
    +
Clothes Reference
    +
Weapon Reference
    +
DWPose
    ↓
IP-Adapter
    ↓
SDXL
```

### 効果

この構成により、以下の問題を大幅に削減できます：

| 問題 | 削減率 |
|------|--------|
| 長袖 → 半袖の変化 | かなり低下 |
| 青服 → 黒服の変化 | かなり低下 |
| 剣 → 短剣の変化 | かなり低下 |
| 髪型変化 | かなり低下 |

---

## 画像ごとの重み（重要）✨

**スプライト生成では、すべての参照画像を同じ強さで与えると逆に不安定になります。**

役割ごとに **重み階層を作る** ことが重要です。

---

## おすすめの優先順位と重み配置

### デフォルト構成（Walk など標準モーション）

```json
{
  "direction": {
    "file": "left_master.png",
    "weight": 0.40
  },
  "face": {
    "file": "face_reference.png",
    "weight": 0.30
  },
  "clothes": {
    "file": "clothes_reference.png",
    "weight": 0.20
  },
  "weapon": {
    "file": "weapon_reference.png",
    "weight": 0.10
  }
}
```

| 要素 | 重み | 理由 |
|------|------|------|
| **方向画像** | 40% | 向きが崩れると致命的（左向き → 正面向きになる） |
| **顔画像** | 30% | 顔が崩れると別人になる |
| **服画像** | 20% | 多少の誤差は許容可能 |
| **武器画像** | 10% | 優先度最低 |

---

## 重みを変更した設定例

### Why この順番か？

```
方向が崩れる  →  左向き が 正面向き に変わる  →  🔴 致命的
    ↓
顔が崩れる  →  別人になる  →  🔴 重要
    ↓
服が崩れる  →  多少の誤差  →  🟡 許容
    ↓
武器が崩れる  →  一番後   →  🟢 低優先度
```

---

## モーション別の最適な重み配置

すべての参照画像に同じ重みを使うのではなく、**モーション種類に応じて最適値が異なります。**

### Walk（歩行）

```json
{
  "direction": 0.40,
  "face": 0.30,
  "clothes": 0.20,
  "weapon": 0.10
}
```

**特徴**：バランス型。向きと顔を重視しつつ、全体的に安定。

---

### Attack（攻撃）

```json
{
  "direction": 0.35,
  "face": 0.25,
  "clothes": 0.15,
  "weapon": 0.25
}
```

**特徴**：武器重視。攻撃モーションでは武器の形状がより重要になるため、重みを上げる。

---

### Cast（魔法・スキル）

```json
{
  "direction": 0.30,
  "face": 0.40,
  "clothes": 0.20,
  "weapon": 0.10
}
```

**特徴**：顔重視。表情やジェスチャーが重要なため、顔の重みを上げる。

---

## KImageGraph への実装方法

### Reference Pack に重みを含める

```json
{
  "motion": "walk",
  "references": [
    {
      "type": "direction",
      "file": "left_master.png",
      "weight": 0.40
    },
    {
      "type": "face",
      "file": "face.png",
      "weight": 0.30
    },
    {
      "type": "clothes",
      "file": "clothes.png",
      "weight": 0.20
    },
    {
      "type": "weapon",
      "file": "sword.png",
      "weight": 0.10
    }
  ]
}
```

### Reference Pack Builder ノードで自動計算

```
Character Master Pack + Motion Type
    ↓
Reference Pack Builder
（モーション種類に応じて重みを自動設定）
    ↓
Multi Reference IP Adapter
    ↓
SDXL
```

---

## 自動調整機構（高度な応用）

**QualityCheck の結果を利用して、重みを動的に調整** することで、さらに安定性を向上させることができます。

### 例1：長袖 → 半袖の変化を検出した場合

初期状態：
```json
{
  "clothes_weight": 0.20
}
```

検出：長袖 → 半袖の変化

自動調整：
```json
{
  "clothes_weight": 0.35
}
```

より強く服の特徴を参照するよう重みを上げる。

---

### 例2：顔崩れを検出した場合

初期状態：
```json
{
  "face_weight": 0.30
}
```

検出：顔が歪んだ / 表情がおかしい

自動調整：
```json
{
  "face_weight": 0.45
}
```

より強く顔の特徴を参照するよう重みを上げる。

---

## Komatta365 推奨設定（実装例）

実際にプロジェクトで採用する場合の推奨設定：

### デフォルト（標準モーション用）

```json
{
  "direction": 0.45,
  "face": 0.30,
  "clothes": 0.15,
  "weapon": 0.10
}
```

方向と顔を強くして基本的な安定性を確保。

---

### Attack 系（攻撃モーション用）

```json
{
  "direction": 0.35,
  "face": 0.25,
  "clothes": 0.15,
  "weapon": 0.25
}
```

武器の重みを大幅に上げて、攻撃の特徴を強調。

---

### 実装上のポイント

**Reference Pack Builder ノードでモーション種類に応じて重みを自動切り替える** のが最も実用的です。

```
ReferencePack Builder
    ├─ motion_type (Walk / Attack / Cast / ...)
    ├─ Character Master Pack
    └─ Weight Profile (モーション別重み設定)
           ↓
        重みを自動計算・設定
           ↓
    Multi Reference IP Adapter へ渡す
```

---

## まとめ：重み設定のベストプラクティス

| ポイント | 説明 |
|---------|------|
| **全て同じ重み ✗** | 不安定になるため非推奨 |
| **階層化 ✓** | 役割に応じた重みを設定 |
| **モーション別 ✓** | Attack と Walk で異なる重みを使用 |
| **自動調整 ✓** | QualityCheck と組み合わせてさらに安定化 |
| **Reference Pack ✓** | JSON で重み設定を管理すると扱いやすい |

---

## 袖の一貫性を最優先にした設定（スプライト用途向け）✨

**スプライト用途では、顔の一致率よりも「服の一致率」、特に袖・肩・襟の一致率を優先** した方がアニメーションの違和感が大きく減ります。

---

## 通常の重み配置 vs 袖固定モード

### 通常（バランス重視）

```
方向（40%）> 顔（30%）> 服（20%）> 武器（10%）
```

### 袖固定モード（衣装一貫性重視）✓ 推奨

```
服（50%）> 方向（25%）> 顔（20%）> 武器（5%）
```

---

## 袖固定モードの推奨設定

### シンプル版

```json
{
  "clothes": 0.50,
  "direction": 0.25,
  "face": 0.20,
  "weapon": 0.05
}
```

**目的**：長袖⇔半袖問題を防ぎ、服装の一貫性を最優先。

---

### 高度な版：袖・襟・肩の専用参照画像

通常は服全体を参照しますが、**袖・襟・肩を切り出した専用参照画像** を別途作成することで、さらに精度を向上させることができます。

#### ファイル構成

```
left_master.png           # 方向マスター
clothes_sleeve.png        # 袖・襟・肩の専用参照
face.png                  # 顔
weapon.png                # 武器
```

#### 重み配置

```json
{
  "sleeve_reference": 0.40,
  "direction_reference": 0.25,
  "face_reference": 0.20,
  "weapon_reference": 0.05,
  "other_clothes_reference": 0.10
}
```

| 参照画像 | 重み | 説明 |
|---------|------|------|
| **袖専用参照** | 40% | 最優先。長袖・半袖を固定 |
| **方向マスター** | 25% | 向きの安定性 |
| **顔参照** | 20% | 顔の一貫性 |
| **その他衣装** | 10% | ベストの色・パターンなど |
| **武器参照** | 5% | 最低優先度 |

---

## Character Definition に固定属性を追加

**キャラクターの属性として「袖の種類」を固定** することで、生成時に自動的に特定の袖タイプが選ばれるようにします。

```json
{
  "character_id": "hero_01",
  "name": "主人公",
  "locked_features": {
    "sleeve_type": "long_sleeve",
    "coat_type": "long_coat"
  }
}
```

これにより、生成システムが常に指定された袖・コートタイプを参照するようになります。

---

## Prompt の補強

### Positive Prompt（強調すべき要素）

```
long sleeve,
long sleeve coat,
full sleeve,
consistent costume,
same outfit,
same clothing design,
identical sleeves,
long sleeves throughout
```

### Negative Prompt（排除すべき要素）

```
short sleeve,
sleeveless,
bare arms,
rolled sleeves,
different outfit,
costume variation,
changing sleeves,
mixed sleeve lengths
```

---

## 自動検査：Sleeve Validator の実装

生成後に **袖の一貫性を自動チェック** し、問題があれば自動再生成します。

### フロー

```
Generated Frame
    ↓
Sleeve Validator
（袖の種類・色を検査）
    ↓
Result:
{
  "expected": "long_sleeve",
  "detected": "short_sleeve",
  "confidence_score": 0.32
}
```

### 判定基準

```json
{
  "minimum_score": 0.90
}
```

**スコア < 0.90** の場合は自動再生成。

---

## 推奨される実装構成（最高精度版）

Komatta365 が採用する構成：

```
Character LoRA
    ↓
Reference Pack Builder
    ├─ Sleeve Reference        (weight=0.40)
    ├─ Coat Reference          (weight=0.10)
    ├─ Direction Master        (weight=0.25)
    ├─ Face Reference          (weight=0.20)
    └─ Weapon Reference        (weight=0.05)
    ↓
Multi Reference IP-Adapter
    ↓
DWPose
    ↓
Image Generator (SDXL)
    ↓
Sleeve Validator
    │
    ├─ ✅ Pass（score >= 0.90）
    │   ↓
    │  Output Frame
    │
    └─ ❌ Fail（score < 0.90）
        ↓
       Retry Generation
        ↓
       (再度 IP-Adapter から開始)
```

### 各ステップの役割

| ステップ | 役割 |
|---------|------|
| **Character LoRA** | キャラクターのベース特徴を提供 |
| **Reference Pack Builder** | 袖固定モードで重みを自動構築 |
| **Sleeve Reference** | 袖・襟・肩の形状を最優先で制御 |
| **IP-Adapter** | 複数参照画像を融合 |
| **DWPose** | ポーズの一貫性を確保 |
| **Sleeve Validator** | 袖の種類・色が一致しているか検査 |
| **自動再試行** | スコア不足の場合は再生成 |

---

## 袖固定モードのデフォルト設定

**スプライト用途では以下を推奨デフォルト** として採用することをお勧めします。

```json
{
  "mode": "sleeve_locked",
  "character_locked_features": {
    "sleeve_type": "long_sleeve",
    "coat_type": "long_coat"
  },
  "weight_profile": {
    "sleeve_reference": 0.40,
    "direction_reference": 0.25,
    "face_reference": 0.20,
    "other_clothes_reference": 0.10,
    "weapon_reference": 0.05
  },
  "validator": {
    "enabled": true,
    "minimum_confidence": 0.90,
    "auto_retry": true,
    "max_retries": 3
  }
}
```

---

## まとめ：袖固定モードの効果

| 項目 | 効果 |
|------|------|
| **長袖 ⇔ 半袖の揺らぎ** | ✅ 大幅に削減 |
| **衣装色の変化** | ✅ 軽減 |
| **袖・襟・肩の形状変化** | ✅ 最小化 |
| **アニメーション違和感** | ✅ 大きく改善 |
| **顔の一貫性** | ⚠️ やや低下（代わりに衣装優先） |

---

## 再生成なしで袖の崩れを直す方法（後処理）

**生成後にValidator で袖の不一致が検出された場合、再生成ではなく後処理で直す方法** があります。

---

## 発想の転換

### ❌ 従来の方法（再生成）

```
生成
  ↓
袖の不一致発見
  ↓
再生成
  ↓
（処理が重い）
```

### ✅ 新しい方法（後処理）

```
生成
  ↓
袖だけ補正
  ↓
完成
  ↓
（軽い・高速）
```

---

## 方法1：袖領域だけ転写（シンプル）

### 概要

基準フレームの袖領域を抽出して、問題フレームに貼り付ける方法。

### 処理フロー

**基準フレーム（正常）**
```
walk_01（長袖）
```

**問題フレーム（エラー）**
```
walk_03（半袖に崩れた）
```

### 処理ステップ

#### 1. 基準フレームから袖マスク抽出

```
walk_01
  ↓
Sleeve Extractor
  ↓
left_sleeve.png
right_sleeve.png
```

#### 2. 問題フレームに貼り付け

```
walk_01（長袖）
    +
walk_03（半袖）
    ↓
Sleeve Compositor
    ↓
walk_03（長袖に修正）
```

### メリット・デメリット

| 項目 | 説明 |
|------|------|
| ✅ **速度** | 極めて高速 |
| ✅ **品質** | 基準フレームと完全に一致 |
| ⚠️ **課題** | ポーズが異なると違和感（肘の角度など） |

---

## 方法2：骨格追従スリーブ（推奨）✨

**最も安定した方法。** 肩→肘→手首のボーンに沿って袖を変形させるため、ポーズが異なっていても違和感がありません。

### 処理フロー

```
Reference Frame（基準画像）
  ├─ left_arm
  └─ right_arm
  
      ↓
      
DWPose 抽出
  ├─ shoulder position
  ├─ elbow position
  └─ wrist position
      
      ↓
      
Sleeve Mesh Warping
（袖テクスチャを腕骨格へワープ）
      
      ↓
      
Corrected Frame（修正完了）
```

### 具体的な処理

#### ステップ1：基準フレームから袖テクスチャ抽出

```
walk_01（基準）
  ↓
Sleeve Mesh Extractor
  ↓
{
  "left_sleeve_texture": "...",
  "right_sleeve_texture": "...",
  "mesh_data": "..."
}
```

#### ステップ2：現在フレームの腕骨格を取得

```
walk_03（問題フレーム）
  ↓
DWPose
  ↓
{
  "left_shoulder": [x1, y1],
  "left_elbow": [x2, y2],
  "left_wrist": [x3, y3],
  "right_shoulder": [x4, y4],
  ...
}
```

#### ステップ3：テクスチャを腕骨格にワープして貼り付け

```
Sleeve Texture + Arm Skeleton
  ↓
Mesh Warp Transform
（肩→肘→手首に沿って変形）
  ↓
walk_03_corrected（完成）
```

### メリット

| 項目 | 説明 |
|------|------|
| ✅ **どんなポーズでも適用可** | 腕の角度が変わってもフィット |
| ✅ **自然な見た目** | ボーンに追従するため違和感なし |
| ✅ **高速** | 再生成よりはるかに軽い |
| ✅ **再利用可能** | 1つの袖テクスチャで複数フレームに対応 |

---

## 方法3：部位ライブラリ方式

**ゲーム開発的アプローチ。** キャラクターを部位ごとにレイヤー管理する方法。

### Character 定義

```json
{
  "character_id": "hero_01",
  "parts": {
    "hair": "hair_model.png",
    "face": "face_model.png",
    "torso": "torso_model.png",
    "sleeve": "long_sleeve_model.png",
    "pants": "pants_model.png",
    "boots": "boots_model.png",
    "weapon": "sword_model.png"
  }
}
```

### 生成後の処理

```
Generated Frame
  ├─ Body
  ├─ Face
  └─ Clothes (←ここが不正)
        ↓
Replace with Library Part
        ↓
Long Sleeve Part from Library
        ↓
Composed Frame（完成）
```

### メリット

- キャラクターパーツを統一管理可能
- 部位ごとに品質管理が容易

### デメリット

- 生成品質に依存（生成フレームの腕領域が大きく異なると使えない）

---

## 方法4：セグメント再描画（高度）

**最新のAI Inpaint 技術を活用した方法。** 再生成ではなく、指定領域だけを描き直します。

### 処理フロー

```
Generated Frame
  ↓
Body Segmentation
（上腕・前腕・袖を分離）
  ↓
Sleeve Mask Generation
  ↓
Inpaint（袖領域だけ再描画）
  ├─ Prompt: "long sleeve"
  └─ Negative: "short sleeve, bare arms"
  ↓
Refined Frame（完成）
```

### メリット

- 全身再生成が不要
- 袖だけ自然に描き直される

### デメリット

- Inpaint モデルの品質に左右される
- 方法1-3より処理時間がかかる

---

## KImageGraph 向けノード設計

Komatta365 が採用する専用ノード群：

### 基本構成

```
Reference Frame
    │
    ▼
Sleeve Extractor
（基準フレームから袖テクスチャ + メッシュ抽出）
    │
    ▼
Sleeve Asset
（再利用可能な袖リソース）


Generated Frame
    │
    ▼
Sleeve Corrector
（骨格追従でワープ・貼付）
    │
    ▼
Corrected Frame
（修正完了）
```

---

## Sleeve Corrector ノード（詳細）

### 入力

```json
{
  "source_sleeve": "walk_01",
  "target_frame": "walk_03",
  "skeleton": "walk_03_skeleton.json",
  "method": "bone_warp"
}
```

| 項目 | 説明 |
|------|------|
| `source_sleeve` | 基準フレーム（正常な袖） |
| `target_frame` | 修正対象フレーム（崩れた袖） |
| `skeleton` | ターゲットの腕骨格データ |
| `method` | 処理方式（bone_warp / simple_paste など） |

### 内部処理

```
Input (source_sleeve, target_frame, skeleton)
  ↓
1. Extract Sleeve Mesh from source_sleeve
  ├─ Shoulder position
  ├─ Elbow position
  └─ Wrist position
  ↓
2. Load Target Skeleton
  ├─ target_shoulder
  ├─ target_elbow
  └─ target_wrist
  ↓
3. Mesh Warping & Transformation
  └─ 肩→肘→手首へテクスチャを変形
  ↓
4. Blend & Composite
  └─ 修正フレームに合成
  ↓
Output: walk_03_fixed
```

### 出力

```json
{
  "output_frame": "walk_03_corrected.png",
  "sleeve_confidence": 0.98,
  "blend_quality": 0.95
}
```

---

## 複合補正ノード群（最終形態）

スプライト生成の完全な後処理パイプライン：

```
Generated Frame
    │
    ├─ Face Corrector
    │  （顔の位置・表情を補正）
    │
    ├─ Sleeve Corrector ← 最優先
    │  （袖を骨格追従で補正）
    │
    └─ Weapon Corrector
       （武器の位置・形状を補正）
           ↓
        Composite
           ↓
    Final Frame（完成）
```

### なぜ Sleeve Corrector を最優先にするのか？

| 理由 | 重要度 |
|------|--------|
| **視覚的違和感** | 袖の変化が最も目立つ |
| **アニメーション連続性** | 袖がチカチカすると致命的 |
| **難易度** | 顔より袖の方が補正しやすい |

---

## 推奨：骨格追従スリーブ方式

**Komatta365 が最終的に採用する設定：**

```json
{
  "post_processing": {
    "enabled": true,
    "pipeline": [
      {
        "type": "sleeve_corrector",
        "priority": 1,
        "method": "bone_warp",
        "reference_frame": "walk_01",
        "enable_mesh_warping": true,
        "bone_joints": ["shoulder", "elbow", "wrist"],
        "quality": "high"
      },
      {
        "type": "face_corrector",
        "priority": 2,
        "method": "landmark_align"
      },
      {
        "type": "weapon_corrector",
        "priority": 3,
        "method": "simple_paste"
      }
    ]
  }
}
```

### 利点

```
✅ 再生成なし：超高速
✅ 品質維持：基準フレームと完全一致
✅ ポーズ対応：どんな腕の角度でもフィット
✅ フレーム統一：1基準フレームで全アニメーション対応
✅ 軽量処理：GPU負荷が低い
```

---

## 実装フロー（完全版）

```
Character Master Pack + Reference Frame
    │
    ▼
Sleeve Extractor
（基準から袖テクスチャ + メッシュ抽出）
    │
    ▼
Sleeve Asset Library
（再利用可能なリソース）


AI Image Generator
    │
    ├─ Prompt + Reference Pack
    ├─ IP-Adapter
    └─ SDXL
        ↓
    Generated Frame
        ↓
    Sleeve Validator
    （スコアチェック）
        │
        ├─ ✅ Score >= 0.90
        │   └─ Output
        │
        └─ ❌ Score < 0.90
            ↓
        Sleeve Corrector
        （骨格追従でワープ修正）
            │
            ├─ DWPose extraction
            ├─ Mesh warping
            └─ Composite
            ↓
        Corrected Frame
            ↓
            ✅ Output
```

---

## まとめ：再生成 vs 後処理

| 方法 | 処理時間 | 品質 | 安定性 | 推奨度 |
|------|---------|------|--------|--------|
| **再生成** | 遅い | 高 | 中 | ⚠️ 最終手段 |
| **方法1：袖転写** | 極速 | 中 | 低 | ⭐⭐ |
| **方法2：骨格追従** | 高速 | 高 | 高 | ⭐⭐⭐⭐⭐ |
| **方法3：部位ライブラリ** | 高速 | 中 | 中 | ⭐⭐⭐ |
| **方法4：Inpaint再描画** | 中速 | 高 | 中 | ⭐⭐⭐ |

**スプライト用途では、基準フレームの長袖テクスチャを現在フレームの腕骨格へワープして貼り付ける「方法2：骨格追従スリーブ」が最も安定かつ実用的です。** 🎯

### 推奨アプローチ

**「IP-Adapter に 1 枚だけ渡す」のではなく、「Reference Pack として複数画像を扱う設計」** が、スプライト生成システムに最も相性が良いです。

### 理由

1. **参照情報の豊富さ** - 複数画像により特徴がより正確に抽出される
2. **一貫性の維持** - 顔、髪、服、装備の変化を最小限に抑える
3. **スケーラビリティ** - 新しい要素（帽子、アクセサリーなど）が追加しやすい
4. **メンテナンス性** - 参照画像を個別に更新できる
5. **重み調整** - モーション別に最適な重み配置が可能

### 実装上の注意点

- Direction Master は必ず含める（向き統一のため）
- DWPose は重要（ポーズ変化の抑止）
- 参照画像の品質が高いほど良い結果
- 参照画像のサイズはできるだけ統一する
- **各参照画像に適切な重みを設定する（全て同じ重みは ✗）**

---

## まとめ

| パターン | 複雑度 | 効果 | 推奨 |
|---------|--------|------|------|
| パターン1（単一参照） | 低 | 低 | 初期テスト |
| パターン2（段階的IP-Adapter） | 高 | 中 | 非推奨 |
| パターン3（Reference Pack） | 中 | 高 | ✓ 推奨 |

**スプライト生成では、パターン3（Reference Pack）を採用することをお勧めします。**
