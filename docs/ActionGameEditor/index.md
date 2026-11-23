# ActionGameEditor

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

ActionGameEditor は、アクションゲームの制作を支援するための統合エディタです。ゲームロジック、キャラクター制御、イベント管理などをノードベースで直感的に設計できます。

## TODO リスト

ActionGameEditor 開発の進捗：

- [x] アーキテクチャ設計
- [x] ノードベースロジックエディタ
- [x] キャラクター定義システム
- [x] 状態管理機能
- [x] 敵AI制御
- [x] 衝突・ダメージシステム
- [ ] ビジュアルエディタUI改善
- [ ] アニメーション連携
- [ ] イベントシステム拡張
- [ ] マルチプレイ対応
- [ ] ゲームバランス計測ツール
- [ ] デバッグビジュアライザー

## 目次

- [概要](#概要)
- [主な機能](#主な機能)
- [ゲームロジックノード](#ゲームロジックノード)
- [キャラクター制御](#キャラクター制御)
- [使用例](#使用例)
- [出力形式](#出力形式)
- [ワークフロー](#ワークフロー)
- [推奨される用途](#推奨される用途)
- [関連ドキュメント](#関連ドキュメント)

---

## 概要

ActionGameEditor は、以下の機能を提供します：

- **ノードベースロジックエディタ** - ゲームロジックを直感的に設計
- **キャラクター制御** - プレイヤーと敵の動作を定義
- **状態管理** - キャラクターの状態遷移を視覚的に管理
- **アニメーション連携** - スプライトアニメーションの自動制御
- **イベント・衝突判定** - インタラクティブなゲーム要素の設定
- **メタデータ出力** - JSON形式でのゲーム設定エクスポート

## 主な機能

### キャラクター定義

プレイヤーと敵キャラクターの基本属性：

```
キャラクター定義ノード
├─ ID・名前設定
├─ 初期位置・向き
├─ サイズ・スケール
├─ 衝突判定設定
└─ ステータス（HP、攻撃力など）
```

### 状態管理

キャラクターの状態遷移を定義：

```
状態ノード
├─ Idle（待機）
├─ Run（走行）
├─ Jump（ジャンプ）
├─ Attack（攻撃）
├─ Damage（ダメージ）
├─ Dead（死亡）
└─ カスタム状態
```

### 入力制御

プレイヤー入力の処理：

```
入力ノード
├─ キーボード入力
├─ ゲームパッド入力
├─ タッチ入力
└─ 入力バッファリング
```

### 物理演算

ゲーム内の物理的な動作：

```
物理ノード
├─ 重力
├─ 速度・加速度
├─ 衝突応答
└─ 移動制限
```

### 敵AI制御

敵キャラクターの自動動作：

```
AI ノード
├─ パトロール（巡回）
├─ 追跡（プレイヤー追従）
├─ 攻撃判定
├─ 逃走
└─ 連携動作
```

## ゲームロジックノード

### キャラクター定義ノード

```json
{
  "type": "character",
  "id": "player_01",
  "name": "主人公",
  "spriteFile": "player.png",
  "initialPosition": {
    "x": 100,
    "y": 200
  },
  "size": {
    "width": 64,
    "height": 64
  },
  "stats": {
    "maxHp": 100,
    "attack": 20,
    "defense": 5,
    "speed": 200
  },
  "collider": {
    "type": "box",
    "width": 50,
    "height": 60,
    "offsetX": 7,
    "offsetY": 4
  }
}
```

### 状態遷移ノード

```json
{
  "type": "state",
  "states": [
    {
      "name": "idle",
      "animation": "idle",
      "duration": null,
      "transitions": [
        {
          "event": "moveInput",
          "targetState": "run"
        },
        {
          "event": "jumpInput",
          "targetState": "jump"
        },
        {
          "event": "attackInput",
          "targetState": "attack"
        }
      ]
    },
    {
      "name": "run",
      "animation": "run",
      "speed": 200,
      "transitions": [
        {
          "event": "noInput",
          "targetState": "idle"
        },
        {
          "event": "jumpInput",
          "targetState": "jump"
        }
      ]
    },
    {
      "name": "jump",
      "animation": "jump",
      "duration": 1.2,
      "velocityY": -400,
      "transitions": [
        {
          "event": "landGround",
          "targetState": "idle"
        }
      ]
    },
    {
      "name": "attack",
      "animation": "attack",
      "duration": 0.6,
      "damageFrame": 0.3,
      "transitions": [
        {
          "event": "attackEnd",
          "targetState": "idle"
        }
      ]
    }
  ]
}
```

### 敵AI制御ノード

```json
{
  "type": "enemyAI",
  "id": "enemy_goblin",
  "aiType": "patrol_and_chase",
  "patrolPoints": [
    {"x": 100, "y": 300},
    {"x": 300, "y": 300},
    {"x": 500, "y": 300}
  ],
  "patrolSpeed": 100,
  "detectionRange": 200,
  "chaseSpeed": 150,
  "attackRange": 50,
  "attackCooldown": 1.5,
  "fleeHealth": 20,
  "fleeSpeed": 200
}
```

### 衝突・ダメージノード

```json
{
  "type": "collision",
  "collisions": [
    {
      "objectA": "player_01",
      "objectB": "enemy_goblin",
      "event": "onHit",
      "damage": 10,
      "knockback": 150
    },
    {
      "objectA": "player_01_attack",
      "objectB": "enemy_goblin",
      "event": "onAttackHit",
      "damage": 20,
      "knockback": 200
    }
  ]
}
```

## キャラクター制御

### プレイヤー制御フロー

```
入力検出
    ↓
状態判定
    ↓
移動・ジャンプ計算
    ↓
衝突判定
    ↓
アニメーション更新
    ↓
画面描画
```

### 敵AI制御フロー

```
敵初期化
    ↓
AIループ
  ├─ プレイヤー検出
  ├─ 移動・攻撃判定
  ├─ 状態更新
  └─ アニメーション更新
    ↓
画面描画
```

## 使用例

### シンプルなアクションゲーム

```
プレイヤー定義
    ↓
入力制御設定
    ↓
敵AI定義
    ↓
衝突・ダメージ設定
    ↓
ゲームロジック出力
```

### ボスバトル

```
プレイヤー定義
    ↓
ボス敵定義
    ↓
複雑な敵AI（複数フェーズ）
    ↓
衝突・ダメージ設定
    ↓
特殊効果・演出
    ↓
出力
```

### マルチプレイアクションゲーム

```
複数プレイヤー定義
    ↓
敵グループ定義
    ↓
連携AI制御
    ↓
シングルプレイモード
    ↓
マルチプレイモード
    ↓
出力
```

## 出力形式

### ゲームロジック定義（JSON）

```json
{
  "gameConfig": {
    "title": "Adventure Quest",
    "screenWidth": 1280,
    "screenHeight": 720,
    "gravity": 500,
    "characters": [
      { "$ref": "player_01.json" },
      { "$ref": "enemy_goblin.json" }
    ],
    "levels": [
      {
        "id": "level_01",
        "name": "Forest Stage",
        "enemies": ["enemy_goblin", "enemy_orc"],
        "bossEnemyId": "boss_dragon"
      }
    ]
  }
}
```

### キャラクター定義（JSON）

各キャラクターの詳細設定：

```json
{
  "character": {
    "id": "player_01",
    "name": "主人公",
    "spriteSheet": "player_spritesheet.png",
    "stats": {
      "maxHp": 100,
      "attack": 20,
      "defense": 5,
      "speed": 200
    },
    "states": ["idle", "run", "jump", "attack", "damage"],
    "animations": {
      "idle": {"frames": [0, 1, 2], "speed": 10},
      "run": {"frames": [3, 4, 5, 6], "speed": 15},
      "jump": {"frames": [7], "speed": 1}
    }
  }
}
```

### AI定義（JSON）

敵AI行動の詳細設定：

```json
{
  "ai": {
    "id": "enemy_goblin",
    "name": "ゴブリン",
    "type": "patrol_and_chase",
    "behavior": {
      "patrol": {
        "points": [[100, 300], [300, 300]],
        "speed": 100
      },
      "chase": {
        "detectionRange": 200,
        "speed": 150
      },
      "attack": {
        "range": 50,
        "cooldown": 1.5,
        "damage": 10
      }
    }
  }
}
```

## ワークフロー

### 基本的な開発フロー

```
1. ゲーム仕様書作成
   ↓
2. キャラクター定義
   ↓
3. ゲームロジック構築
   ↓
4. 敵AI制御設定
   ↓
5. イベント・衝突判定設定
   ↓
6. ゲームロジック出力
   ↓
7. ゲームエンジンに統合
   ↓
8. テスト・デバッグ
```

### チームでの開発

```
ゲームデザイナー: ゲーム仕様・キャラクター設計
     ↓
プログラマー: ゲームロジック実装
     ↓
AI開発者: 敵AI制御
     ↓
エフェクト・アニメーター: ビジュアル調整
     ↓
テスター: ゲームバランス調整
```

## 推奨される用途

| 用途 | 説明 |
|------|------|
| アクションゲーム開発 | 2Dアクションゲームの開発を高速化 |
| ゲームロジック設計 | ノードベースで複雑なロジックを視覚的に設計 |
| キャラクター制御 | プレイヤーと敵の動作を直感的に定義 |
| AI制御 | 敵キャラクターの自動動作を設定 |
| ゲームバランス調整 | パラメータを簡単に変更して即座にテスト |

## 関連ドキュメント

- [DesignForApplications 総合ガイド](index.md)
- [TypeScript + Rust + Tauri ハイブリッドアーキテクチャ](typescript-react-rust-wasm-tauri/index.md)
- [WebBrowser ゲーム開発](typescript-react-rust-wasm-tauri/WebBrowserGameDevelopment.md)
- [2DSpriteGraph - スプライト管理](2DSpriteGraph/index.md)
- [SFXGraph - 効果音管理](SFXGraph/index.md)
- [MusicGraph - 音楽管理](MusicGraph/index.md)
