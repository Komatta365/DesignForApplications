# SFXGraph - TODO

SFXGraph は、ゲーム効果音制作のためのノードベースオーディオエディタです。

## 進捗状況

### 進行中 🔄

- [ ] 画像認識 (ONNX Runtime)
  - [ ] CLIP (意味理解)
  - [ ] YOLOv8 (物体検出)
  - [ ] BLIP (説明文生成)
- [ ] オーディオエフェクト実装
- [ ] リアルタイムプレビュー機能

### 予定 📋

- [ ] サンプルを作成
  - [ ] asset/sample/以下にサンプルを作成
  - [ ] サンプルから作成メニューを追加する
- [ ] アドバンスド効果音ライブラリ
- [ ] サンプル波形管理
- [ ] ユーザープリセット保存機能
- [ ] パフォーマンス最適化
- [ ] ユーザーテスト実施

### Event・Blueprint 設計と実装 🏗️

#### Phase 1: スキーマ定義と基本実装

- [ ] Sound Scene JSON スキーマ設計
  - [ ] Sound Scene Schema v1.0 定義（JSON Schema 2020-12）
  - [ ] Scene・Actor・Object 型定義
  - [ ] Sound Scene パッケージ (@ssfx/sound-scene-types)
  - [ ] Vision AI 出力との互換性確保
  - [ ] ユニットテスト（Sound Scene validation）

- [ ] Sound Events JSON スキーマ設計
  - [ ] Sound Events Schema v1.0 定義（JSON Schema 2020-12）
  - [ ] ambientEvents / actionEvents 分離
  - [ ] blueprintHint フィールド対応
  - [ ] Sound Events パッケージ (@ssfx/sound-events-types)
  - [ ] ユニットテスト（Sound Events validation）

- [ ] Event JSON スキーマの確定
  - [ ] Event JSON Schema v1.0 定義（JSON Schema 2020-12）
  - [ ] Event型定義パッケージ (@ssfx/event-types)
  - [ ] 53+ Event type の標準化（11カテゴリ分類）
  - [ ] ParameterMapping の実装（intensity→Gain等）
  - [ ] ユニットテスト（Event validation）

- [ ] Blueprint JSON スキーマ設計
  - [ ] Blueprint Schema v1.0 定義（JSON Schema 2020-12）
  - [ ] Node・Connection・ParameterMapping 型定義
  - [ ] Blueprint パッケージ (@ssfx/blueprint-types)
  - [ ] loopable、estimatedCost 等メタデータ対応
  - [ ] ユニットテスト（Blueprint validation）

- [ ] NodeGraph JSON スキーマ設計
  - [ ] NodeGraph Schema v1.0 定義（JSON Schema 2020-12）
  - [ ] NodeGraph型定義パッケージ (@ssfx/nodegraph-types)
  - [ ] ユニットテスト（NodeGraph validation）

#### 最終的な 5層構造設計

**5層構造で最も保守しやすい設計:**

```
Layer 1: Scene JSON (Vision/LLM出力)
  → 環境情報のみ

Layer 2: Event JSON (Skill出力)
  → 音響イベント定義（ルール）

Layer 3: Blueprint JSON ⭐（Git管理テンプレート）
  → ノード構造テンプレート
  → Copilot が生成すべき唯一の形式
  → 再利用可能（50+ テンプレート）

Layer 4: NodeGraph JSON ⭐（実行時インスタンス）
  → 実行可能グラフ
  → メモリ内生成（一時的）
  → 完全に確定的

Layer 5: Audio (WAV/OGG出力)
  → 波形出力
```

- [ ] NodeGraph Schema v1.0 完全実装
  - [ ] instanceId（グラフインスタンスID）
  - [ ] blueprintId（元テンプレート参照）
  - [ ] sourceEventIds（生成元Event ID群）
  - [ ] Node.runtime（state, cpuCost, voiceCount, lastError）
  - [ ] Node.ui（x, y, width, height, collapsed, color）
  - [ ] Connection.id（接続の一意ID）
  - [ ] Connection.enabled（接続の有効化/無効化）

- [ ] Rain NodeGraph 実例
  - [ ] 完全なJSON サンプル（4ノード・3接続）
  - [ ] Event.intensity → Gain.gain の変換例
  - [ ] UI配置情報付き

- [ ] Blueprint vs NodeGraph の差分テーブル
  - [ ] 目的・Event反映・Node ID・パラメータ・UI位置
  - [ ] 実行状態・ランタイム情報・保管場所・再利用

- [ ] 層ごとの責務の最終明確化
  - [ ] Scene: Vision/LLM入力 (△再現性)
  - [ ] Event: ルールベース変換 (✅再現性)
  - [ ] Blueprint: テンプレート (✅再現性, Git管理)
  - [ ] NodeGraph: 実行インスタンス (✅再現性, メモリ内)
  - [ ] Audio: 波形出力 (✅再現性)

#### NodeGraph の完全な実例

**実運用シーン: 画像解析「雨の森を少女が走る」**

- [ ] 複雑なシーン解析結果の完全NodeGraph例
  - [ ] 11ノード（NoiseGenerator×2, SamplePlayer×2, LowPass, Mixer×3, Reverb, Output）
  - [ ] 11接続で構成
  - [ ] ambientEvents（rain, river, forest）とactionEvents（footstep, breathing）の複合

- [ ] NodeGraph構造設計
  - [ ] Ambient Path: 環境音の合成経路
  - [ ] Action Path: アクション音の合成経路
  - [ ] Master Path: 環境音とアクション音の最終統合
  - [ ] UI配置情報（x, y, width, height）の完全化

- [ ] React Flow / Rete.js への直接展開方法
  - [ ] NodeGraph.nodes → Nodes の直接変換
  - [ ] NodeGraph.connections → Edges の直接変換
  - [ ] ドラッグ可能なノードエディタUI実装

- [ ] Audio Engine の段階的実装計画
  - [ ] Phase 1（コアノード）: Output, Gain, Mixer, SamplePlayer, NoiseGenerator
    - → 85%のシーン実行可能
  - [ ] Phase 2（基本DSP）: LowPass, HighPass, BandPass, Reverb, Envelope
    - → 95%のシーン実行可能
  - [ ] Phase 3（高度な機能）: LFO, EQ, Compressor, Distortion
    - → 99%のシーン実行可能
  - [ ] Phase 4（拡張）: Delay, Panner, Oscillator

**4層構造の統一設計:**

```
Layer 1: Scene JSON
  → Vision AI / LLM による画像・テキスト解析

Layer 2: Event JSON
  → SoundEvent生成 Skill による ルールベース変換

Layer 3: Blueprint JSON ⭐（最も重要）
  → ノード構造テンプレート（Git 管理）
  → Modifier（Event → Node Parameter マッピング）
  → Port ベース設計（audio, control, trigger, event）
  → UI情報＋ Bypass フラグ対応

Layer 4: NodeGraph JSON
  → 実行用インスタンス（runtime 生成、メモリ内）
```

**統合 Blueprint Schema v1 の特徴:**

- [ ] Port ベース設計の完全実装
  - [ ] inputPorts[] / outputPorts[] で明示的に定義
  - [ ] signalType: audio, control, trigger, event の4種類
  - [ ] multiple フラグで多重接続対応
  - [ ] 型安全性により future 対応（LFO・MIDI・Automation等）

- [ ] Node メタデータの完全化
  - [ ] type enum（Oscillator, NoiseGenerator, SamplePlayer, Envelope, LFO, フィルタ, Gain, Mixer, Panner, EQ, Compressor, Distortion, Delay, Reverb, Output）
  - [ ] displayName（UI表示用）
  - [ ] ui: { x, y }（ノードエディタ配置）
  - [ ] bypass: boolean（スキップ可能）
  - [ ] parameters: object（静的パラメータ）

- [ ] Connection の厳密化
  - [ ] sourceNode + sourcePort → targetNode + targetPort
  - [ ] Port ID 参照による型安全な接続
  - [ ] signalType マッチング検証

- [ ] Modifier システムの確立
  - [ ] eventField → nodeId.parameter のマッピング
  - [ ] mapping: linear, log, curve, enum
  - [ ] min, max による値域制約
  - [ ] curvePoints による非線形マッピング

**サンプル実装:**

- [ ] Rain Blueprint v1（完全版）
  - [ ] 4ノード（Noise → LowPass → Gain → Output）
  - [ ] 3接続（audio信号フロー）
  - [ ] intensity modifier（Event.intensity → Gain.gain）
  - [ ] UI配置情報付き

- [ ] Footstep Blueprint 系
  - [ ] footstep_grass_v1
  - [ ] footstep_metal_v1
  - [ ] footstep_stone_v1

**4層統合の最終検証:**

- [ ] Schema 間の互換性確認
  - [ ] Event.type → Blueprint.id の対応関係
  - [ ] Blueprint.modifiers → NodeGraph.nodes パラメータへの正確な変換
  - [ ] SignalType の一貫性（audio→audio のみ接続等）

- [ ] 層間の責務明確化
  - [ ] Scene: 環境情報のみ（Vision/LLM出力）
  - [ ] Event: 音響イベント定義（Skill output）
  - [ ] Blueprint: ノード構造テンプレート（Git 管理）
  - [ ] NodeGraph: 実行可能グラフ（runtime 生成）

- [ ] 段階的実装計画との整合性
  - [ ] Phase 1: Schema定義完了 ✅
  - [ ] Phase 2: Resolver 実装開始
  - [ ] Phase 3: Graph Builder 実装
  - [ ] Phase 4: 統合テスト

#### Phase 2: Template DB と Resolver 実装

- [ ] SoundEvent生成 Skill 実装（Skill中核）
  - [ ] 入力スキーマ実装（Sound Scene JSON Schema）
  - [ ] 出力スキーマ実装（Sound Events JSON Schema）
  - [ ] JSON Schema validator 統合
  - [ ] 6つのルールテーブル実装
    - [ ] Object → Event マッピング
    - [ ] Action → Event マッピング
    - [ ] Weather → Event マッピング
    - [ ] Surface → Tag マッピング
    - [ ] Importance テーブル（Actor別）
    - [ ] Intensity テーブル（Weather別）
  - [ ] SoundSource Resolver 実装
    - [ ] Actor → SoundSource 変換
    - [ ] Object → SoundSource 変換
  - [ ] Surface Resolver 実装
    - [ ] surface タイプ → footstep タグ変換
  - [ ] Weather Resolver 実装
    - [ ] weather → intensity 推定
  - [ ] Ambient / Action 分離処理
  - [ ] blueprintHint 自動生成
  - [ ] Event ID 生成（一意性確保）
  - [ ] パフォーマンステスト（< 10ms目標）
  - [ ] ユニットテスト
    - [ ] 単一Actor + Object
    - [ ] 複合イベント（複数 actor/object/weather）
    - [ ] 足音テスト（複数surface variant）
    - [ ] 天候テスト（複数intensity）
    - [ ] Edge case（空入力等）

- [ ] Blueprint Template DB 構築
  - [ ] 初期テンプレート作成（20+個）
    - [ ] footstep 系（grass, metal, stone等）
    - [ ] rain/weather 系
    - [ ] impact 系
    - [ ] UI 系
  - [ ] テンプレートバージョン管理
  - [ ] タグベースインデックス構築
  - [ ] blueprintHint マッピング テーブル

- [ ] Blueprint Resolver 実装
  - [ ] blueprintHint 優先検索（高速パス）
  - [ ] Event type + tags による検索ロジック
  - [ ] フォールバック処理（default テンプレート選択）
  - [ ] タグ優先度ベースの選択アルゴリズム
  - [ ] ユニットテスト（複合タグ組み合わせ）

#### Phase 3: Graph Builder 実装

- [ ] Graph Builder エンジン
  - [ ] Blueprint → Node Graph 展開ロジック
  - [ ] Parameter Mapping エンジン（Event値 → ノードパラメータ変換）
  - [ ] 複数Event の Mixer 統合
  - [ ] ノード実行順序最適化（トポロジカルソート）
  - [ ] ユニットテスト（各種Event 組み合わせ）

- [ ] Rust Validator 統合
  - [ ] Node型チェック（enum 許容値のみ）
  - [ ] 接続妥当性チェック（ノード出力 → 入力型チェック）
  - [ ] パラメータ範囲チェック（min/max）
  - [ ] グラフサイクル検出（DAG確認）
  - [ ] ユニットテスト（不正グラフ拒否）

#### Phase 4: Skill 統合と AI補助

- [ ] Vision AI 統合（Scene JSON 生成）
  - [ ] Vision API 選定（GPT-4o / Florence2 / CLIP）
  - [ ] Image → Scene JSON 変換プロンプト設計
  - [ ] Scene 要素抽出（scene, actors, objects, weather, surface）
  - [ ] ユニットテスト（複数画像パターン）

- [ ] Skill_TextToEvent 実装
  - [ ] テキスト解析（LLM ベース）
  - [ ] アクション・属性抽出
  - [ ] Event JSON 生成
  - [ ] ユニットテスト

- [ ] Blueprint 自動生成（Copilot CLI）
  - [ ] 新規 Event type 追加時の Blueprint 生成
  - [ ] Copilot プロンプト設計
  - [ ] 生成後の Rust 検証
  - [ ] Template DB への自動登録

### Skill パイプライン統合 🔗

- [ ] Vision AI → Sound Scene （ステップ1）
  - [ ] Vision API 選定（GPT-4o / Florence2 / CLIP等）
  - [ ] Image → Scene JSON 変換プロンプト設計
  - [ ] Scene 要素抽出
    - [ ] scene.environment（indoor/outdoor等）
    - [ ] scene.weather（rain/storm等）
    - [ ] scene.surface（grass/metal等）
    - [ ] actors[] 抽出（type, action, movement等）
    - [ ] objects[] 抽出（type, state等）
  - [ ] Vision 出力の正規化
  - [ ] ユニットテスト（複数画像パターン）
  - [ ] パフォーマンスベンチマーク（100-500ms目標）

- [ ] 自然言語テキスト → Sound Scene （ステップ1b）
  - [ ] LLM テキスト解析プロンプト設計
  - [ ] Text → Scene JSON 変換
  - [ ] Scene 要素抽出
    - [ ] scene.environment（森、洞窟、戦場等）
    - [ ] scene.weather（雨、嵐、晴れ等）
    - [ ] scene.surface（草、金属、石等）
    - [ ] actors[] 抽出（キャラクタータイプ、アクション等）
    - [ ] objects[] 抽出（物体タイプ、状態等）
  - [ ] テキスト出力の正規化
  - [ ] ユニットテスト（複数テキストパターン）
  - [ ] パフォーマンスベンチマーク（1-3秒目標）

- [ ] Sound Scene → Sound Events （ステップ2：Skill中核）
  - [ ] SoundEvent生成 Skill 実装（推論エンジン）
  - [ ] 6つのルールテーブル実装
  - [ ] SoundSource Resolver
  - [ ] Surface / Weather Resolver
  - [ ] Ambient / Action 分離
  - [ ] blueprintHint 生成
  - [ ] パフォーマンステスト（< 10ms目標）⚡
  - [ ] ユニットテスト（15+ パターン）

- [ ] Sound Events → Blueprint （ステップ3）
  - [ ] Blueprint Resolver 実装
  - [ ] blueprintHint 優先検索
  - [ ] type + tags 組み合わせ検索
  - [ ] フォールバック処理
  - [ ] パフォーマンステスト（1-5ms目標）
  - [ ] ユニットテスト

- [ ] Blueprint → Node Graph （ステップ4）
  - [ ] Graph Builder 実装
  - [ ] Parameter Mapping エンジン
  - [ ] Mixer 統合処理
  - [ ] ユニットテスト（各種Event）

- [ ] Node Graph → Audio （ステップ5）
  - [ ] Audio Engine 実装
  - [ ] リアルタイム波形生成
  - [ ] ユニットテスト

### RAG とテンプレート改善 📚

- [ ] SE テンプレートDB の設計
  - [ ] scene ベースの分類 (forest, dungeon, battlefield, etc)
  - [ ] action ベースの分類 (sword_slash, footstep, impact, etc)
  - [ ] genre ベースの分類 (fantasy, sci-fi, horror, etc)
  - [ ] 推奨ノード構成とパラメータ設定値

- [ ] RAG 検索エンジン
  - [ ] クエリ → テンプレート検索
  - [ ] 複合条件マッチング (scene + action + genre)
  - [ ] フォールバック (不完全マッチ時の代替案)

### 完了 ✅

- [x] アーキテクチャ設計
- [x] ノードベースオーディオエディタUI
- [x] 波形生成基本機能

## 開発リンク

- [SFXGraph](./index.md)
- [画像を解析とノードグラフ作成](./specs/画像を解析とノードグラフ作成.md)
- [自然言語でのノード作成と画像解析からのノード生成Skill](./specs/自然言語でのノード作成と画像解析からのノード生成Skill.md)
- [Eventスキーマを設計](./specs/Eventスキーマを設計.md) - Event・Blueprint・ノード生成パイプライン完全設計
