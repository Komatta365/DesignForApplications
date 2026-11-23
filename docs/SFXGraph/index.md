# SFXGraph

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

SFXGraph は、ゲーム用の効果音（SFX）を効率的に管理・処理するためのノードベースオーディオエディタです。

## TODO リスト

SFXGraph 開発の進捗：

- [x] アーキテクチャ設計
- [x] ノードベースUI
- [x] 音声ファイル管理
- [x] エフェクト処理（リバーブ、EQ、コンプレッサー）
- [x] 波形解析機能
- [x] メタデータ出力
- [ ] リアルタイムプレビュー
- [ ] Advanced エフェクト（ディレイ、ディストーション）
- [ ] 3D オーディオ対応
- [ ] バッチ処理機能
- [ ] スペクトラムアナライザ
- [ ] A/B 比較機能

## 目次

- [概要](#概要)
- [主な機能](#主な機能)
- [音声処理ノード](#音声処理ノード)
- [使用例](#使用例)
- [出力形式](#出力形式)
- [パラメータ設定](#パラメータ設定)
- [推奨される用途](#推奨される用途)
- [関連ドキュメント](#関連ドキュメント)

---

## 概要

SFXGraph は、以下の機能を提供します：

- **ノードベースオーディオエディタ** - 直感的な音声処理グラフ作成
- **効果音管理** - 複数の音声ファイルを統一的に管理
- **リアルタイム処理** - エフェクト適用の即座確認
- **メタデータ出力** - JSON形式での音声情報エクスポート
- **バッチ処理** - 複数の音声ファイルを一括処理

## 主な機能

### 音声ファイル入力

ゲーム用効果音のインポート：

```
入力ノード
├─ WAV/MP3/OGG対応
├─ サンプリングレート取得
├─ ビットデプス自動検出
└─ メタデータ抽出
```

### 音量・タイミング調整

基本的な音声パラメータ設定：

```
パラメータノード
├─ ゲイン（音量調整）
├─ フェードイン/アウト
├─ ディレイ（遅延）
├─ タイミング調整
└─ パン（左右配置）
```

### エフェクト処理

プロフェッショナルなオーディオエフェクト：

```
エフェクトノード
├─ リバーブ（残響）
├─ イコライザー（EQ）
├─ コンプレッサー
├─ ディレイエフェクト
└─ ディストーション
```

### 波形解析

音声の特性解析と最適化：

```
解析ノード
├─ ピークレベル検出
├─ RMS計算
├─ 周波数スペクトラム分析
└─ 自動ノーマライズ
```

## 音声処理ノード

### 入力ノード

```json
{
  "type": "input",
  "file": "explosion.wav",
  "sampleRate": 44100,
  "channels": 2,
  "bitDepth": 16,
  "duration": 2.5
}
```

### ゲイン調整ノード

```json
{
  "type": "gain",
  "value": -3.5,
  "unit": "dB",
  "description": "3.5dB削減"
}
```

### フェード処理ノード

```json
{
  "type": "fade",
  "fadeIn": {
    "duration": 0.1,
    "curve": "linear"
  },
  "fadeOut": {
    "duration": 0.3,
    "curve": "exponential"
  }
}
```

### リバーブエフェクトノード

```json
{
  "type": "reverb",
  "room": 0.5,
  "damping": 0.7,
  "wetLevel": 0.3,
  "dryLevel": 0.7,
  "width": 1.0
}
```

### イコライザーノード

```json
{
  "type": "equalizer",
  "bands": [
    {
      "frequency": 100,
      "gain": -6,
      "Q": 0.7
    },
    {
      "frequency": 2000,
      "gain": 3,
      "Q": 1.0
    },
    {
      "frequency": 8000,
      "gain": 2,
      "Q": 1.5
    }
  ]
}
```

## 使用例

### シンプルなSFX処理

```
効果音読込 → 音量調整 → ノーマライズ → WAV出力
```

### リバーブ付きエフェクト音

```
原音読込
    ↓
ゲイン調整
    ↓
リバーブ適用
    ↓
出力
```

### 複雑な音声処理パイプライン

```
複数音声読込
    ↓
個別エフェクト処理
    ↓
パラメータ調整
    ↓
波形解析・最適化
    ↓
メタデータ生成
    ↓
出力
```

## 出力形式

### 処理済み音声（WAV/MP3）

圧縮形式で出力：

```json
{
  "format": "wav",
  "sampleRate": 44100,
  "bitDepth": 16,
  "channels": 2
}
```

### メタデータ（JSON）

```json
{
  "sfx": [
    {
      "id": "explosion_01",
      "file": "explosion_01.wav",
      "duration": 2.5,
      "category": "explosion",
      "volume": 1.0,
      "pitch": 1.0,
      "loop": false,
      "priority": 128
    },
    {
      "id": "sword_hit",
      "file": "sword_hit.wav",
      "duration": 0.8,
      "category": "sword",
      "volume": 0.8,
      "pitch": 1.0,
      "loop": false,
      "priority": 100
    }
  ]
}
```

### エフェクト設定（JSON）

```json
{
  "effects": {
    "explosion": {
      "reverb": {
        "room": 0.7,
        "wetLevel": 0.4
      },
      "gain": -2
    },
    "sword_hit": {
      "eq": {
        "highFreq": 8000,
        "highGain": 3
      }
    }
  }
}
```

## パラメータ設定

### ゲーム用SFX最適化プリセット

| プリセット | 用途 | 設定値 |
|-----------|------|--------|
| **UI音** | ボタン、メニュー | -6dB、短リバーブ |
| **打撃音** | 剣、拳 | 0dB、高周波ブースト |
| **爆発音** | 爆発、大ダメージ | -3dB、ロング残響 |
| **環境音** | 歩行、風 | -12dB、低周波カット |
| **ボイス** | キャラクター音声 | -2dB、EQ調整 |

## 推奨される用途

| 用途 | 説明 |
|------|------|
| ゲーム効果音管理 | すべてのSFXを統一的に処理・出力 |
| 音声ライブラリ構築 | ゲーム用の効果音ライブラリを自動生成 |
| エフェクト開発 | カスタムエフェクトの設計・実装 |
| 音質最適化 | 配布用に音声を自動最適化 |
| バッチ処理 | 大量の効果音を一括処理・変換 |

## 関連ドキュメント

- [DesignForApplications 総合ガイド](index.md)
- [TypeScript + Rust + Tauri ハイブリッドアーキテクチャ](typescript-react-rust-wasm-tauri/index.md)
- [WebBrowser ゲーム開発](typescript-react-rust-wasm-tauri/WebBrowserGameDevelopment.md)
- [2DSpriteGraph - スプライト管理](2DSpriteGraph/index.md)
- [MusicGraph - 音楽管理](MusicGraph/index.md)
