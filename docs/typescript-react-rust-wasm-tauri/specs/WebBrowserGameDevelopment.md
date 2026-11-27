# Webブラウザでのゲーム開発

## 目次

- [1. 2026年のWebブラウザゲーム開発トレンド](#1-2026年のwebブラウザゲーム開発トレンド)
  - [1.1 WebGPU への移行](#11-webgpu-への移行)
  - [1.2 ゲーム開発スタック（2026年標準）](#12-ゲーム開発スタック2026年標準)
  - [1.3 フロントエンド言語の標準化](#13-フロントエンド言語の標準化)
  - [1.4 エンジンの選択肢](#14-エンジンの選択肢)
  - [1.5 Rust + WASM の重要性](#15-rust--wasm-の重要性)
- [2. WebGPU を活用したレンダリング](#2-webgpu-を活用したレンダリング)
  - [2.1 Three.js + WebGPU の基本セットアップ](#21-threejs--webgpu-の基本セットアップ)
  - [2.2 Compute Shader による物理演算](#22-compute-shader-による物理演算)
  - [2.3 大量オブジェクト描画（インスタンシング）](#23-大量オブジェクト描画インスタンシング)
  - [2.4 パーティクルシステム](#24-パーティクルシステム)
- [3. ブラウザ内 AI の統合](#3-ブラウザ内-ai-の統合)
  - [3.1 CLIP を使った画像認識](#31-clip-を使った画像認識)
  - [3.2 ONNX Runtime Web による推論](#32-onnx-runtime-web-による推論)
  - [3.3 敵AI の実装（ONNX + 意思決定）](#33-敵ai-の実装onnx--意思決定)
  - [3.4 リアルタイム音声認識](#34-リアルタイム音声認識)
- [4. 入力処理と状態管理](#4-入力処理と状態管理)
  - [4.1 キーボード入力の管理](#41-キーボード入力の管理)
  - [4.2 ポインター入力（マウス/タッチ）](#42-ポインター入力マウスタッチ)
  - [4.3 Zustand を使った状態管理](#43-zustand-を使った状態管理)
- [5. 物理演算（Rust + WASM）](#5-物理演算rust--wasm)
  - [5.1 Rust で実装する物理エンジン](#51-rust-で実装する物理エンジン)
  - [5.2 TypeScript から Rust 物理エンジンを呼び出す](#52-typescript-から-rust-物理エンジンを呼び出す)
  - [5.3 ゲームループでの物理統合](#53-ゲームループでの物理統合)
- [6. アセット管理と配信の最適化](#6-アセット管理と配信の最適化)
  - [6.1 推奨アセット形式](#61-推奨アセット形式)
  - [6.2 アセットマネージャー](#62-アセットマネージャー)
  - [6.3 ローディング画面](#63-ローディング画面)
- [7. パフォーマンス最適化](#7-パフォーマンス最適化)
  - [7.1 WebGPU パフォーマンスの最大化](#71-webgpu-パフォーマンスの最大化)
  - [7.2 メモリ管理とキャッシング](#72-メモリ管理とキャッシング)
  - [7.3 フレームレート監視とアダプティブレンダリング](#73-フレームレート監視とアダプティブレンダリング)
  - [7.4 ダイナミックグレードダウン](#74-ダイナミックグレードダウン)
- [8. デバッグと開発支援](#8-デバッグと開発支援)
  - [8.1 ゲームデバッガーUI](#81-ゲームデバッガーui)
  - [8.2 Three.js デバッグヘルパー](#82-threejs-デバッグヘルパー)
  - [8.3 ネットワークプロファイリング](#83-ネットワークプロファイリング)
- [9. クロスプラットフォーム戦略](#9-クロスプラットフォーム戦略)
  - [9.1 Web版とDesktop版（Tauri）の共有コア](#91-web版とdesktop版tauriの共有コア)
  - [9.2 環境固有の処理の分離](#92-環境固有の処理の分離)
- [10. セーブ・ロード機能](#10-セーブロード機能)
  - [10.1 ゲーム状態のシリアライズ（Rust）](#101-ゲーム状態のシリアライズrust)
  - [10.2 セーブUIの実装](#102-セーブuiの実装)
- [11. 推奨事項（2026年標準）](#11-推奨事項2026年標準)
- [12. 技術選定まとめ](#12-技術選定まとめ)
- [13. まとめ](#13-まとめ)
- [14. Webゲームの公開手順](#14-webゲームの公開手順)
  - [14.1 リリース用ビルドを作成](#141-リリース用ビルドを作成)
  - [14.2 公開先を選ぶ](#142-公開先を選ぶ)
  - [14.3 GitHubにアップロード](#143-githubにアップロード)
  - [14.4 社内GitLabへアップロード](#144-社内gitlabへアップロード)
    - [GitLab Pagesとは](#gitlab-pagesとは)
    - [GitHub Pagesとの比較](#github-pagesとの比較)
    - [Webゲームの公開も可能](#webゲームの公開も可能)
    - [Rust + WASMも公開可能](#rust--wasmも公開可能)
    - [社内利用のメリット](#社内利用のメリット)
    - [リポジトリ自体はPrivateでも、Pagesだけ公開できる？](#リポジトリ自体はprivateでもpagesだけ公開できる)
      - [パターン1: リポジトリ Private + Pages Public](#パターン1-リポジトリ-private--pages-public)
      - [パターン2: リポジトリ Private + Pages 社内限定](#パターン2-リポジトリ-private--pages-社内限定)
      - [パターン3: リポジトリ Private + Pages 認証必須](#パターン3-リポジトリ-private--pages-認証必須)
      - [実際の運用例](#実際の運用例)
      - [配信されるフロントエンドコードの注意点](#配信されるフロントエンドコードの注意点)
      - [Webゲーム向けの推奨構成](#webゲーム向けの推奨構成)
    - [事前確認](#事前確認)
  - [14.5 自動デプロイ設定](#145-自動デプロイ設定)
  - [14.6 HTTPS必須](#146-https必須)
  - [14.7 PWA化](#147-pwa化推奨)
  - [14.8 あなたの構成なら](#148-あなたの構成なら)
  - [14.9 プロトタイプ公開の価値](#149-プロトタイプ公開の価値)
- [15. 次のステップ](#15-次のステップ)

---

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

## TODO リスト

ゲーム開発実装の進捗：

- [x] 2026年トレンド分析
- [x] WebGPU 基礎
- [x] AI統合（CLIP/ONNX）
- [x] 物理演算設計
- [x] パフォーマンス最適化戦略
- [x] 公開手順ドキュメント
- [ ] WebGPU ヘルパーライブラリ開発
- [ ] CLIP統合例（完全実装）
- [ ] 音声認識実装
- [ ] ゲームエンジン統合例
- [ ] マルチプレイ対応
- [ ] プレイテスト実施

---

2026年時点で、Webブラウザゲーム開発は大きく進化しました。WebGPUの登場とRust+WASMの成熟により、ブラウザ上でも高性能なゲーム開発が可能になっています。

Webブラウザ上でゲームを開発・実行する場合、以下の構成を採用することで、最先端のパフォーマンスと機能を実現できます。

```text
Frontend
├─ React + TypeScript
├─ Zustand (状態管理)
└─ UI Components

Rendering
├─ Three.js / Babylon.js
└─ WebGPU (高性能描画)

Heavy Processing
├─ Rust + WASM
└─ GPU Physics / AI推論

AI Integration
├─ CLIP解析
├─ ONNX Runtime Web
└─ WebGPU活用
```

このアーキテクチャにより以下が実現できます。

- インストール不要でプレイ可能
- WebGPUによる大規模描画と物理演算
- WASM経由での高速計算処理
- ブラウザ内AIによるNPC制御や画像解析
- クロスプラットフォーム対応
- デスクトップ版（Tauri）への段階的拡張

このドキュメントでは、2026年標準のWebブラウザゲーム開発における実装パターン、WebGPU活用、AI統合、パフォーマンス最適化について説明します。

## 1. 2026年のWebブラウザゲーム開発トレンド

### 1.1 WebGPU への移行

2026年時点で、ブラウザ上の3D描画は大きな転換期を迎えています。

| 技術 | 用途 | 特徴 |
|------|------|------|
| **WebGPU** | 新規開発、高性能向け | 低レイテンシ、GPU計算、将来性 |
| **WebGL** | レガシー、互換性重視 | 広いブラウザサポート |

WebGPUでは以下が実現できるようになりました。

- **Compute Shader**: GPU上での汎用計算
- **GPU物理演算**: 大量パーティクル、布シミュレーション
- **GPUパーティクル**: 数百万個の粒子を同時描画
- **AI推論**: ONNX Runtimeで機械学習モデル実行
- **大量オブジェクト描画**: インスタンシングで効率的に処理

これにより、以前はデスクトップ限定だったゲームジャンルがブラウザで実現可能になっています。

**実現可能なゲームジャンル:**

- 3D RPG
- RTS（リアルタイムストラテジー）
- シミュレーション（気象、物理、経済）
- MMORPG
- 物理演算ベースのパズル

### 1.2 ゲーム開発スタック（2026年標準）

```text
最先端構成:

Frontend
  React + TypeScript + Zustand

Rendering
  Three.js / Babylon.js + WebGPU

Computing
  Rust → WASM
  
AI
  CLIP + ONNX Runtime Web + WebGPU
  
Asset Format
  glTF + KTX2 + WebP
```

**各レイヤーの役割:**

| レイヤー | 技術 | 責務 |
|---------|------|------|
| UI/State | React + Zustand | 画面描画、状態管理 |
| 3D描画 | Three.js + WebGPU | ジオメトリ、ライティング、シェーダー |
| 物理/AI | Rust + WASM | 高速計算、AI推論 |
| アセット | glTF/KTX2 | モデル、テクスチャ配信 |

### 1.3 フロントエンド言語の標準化

JavaScriptからTypeScriptへの移行が完全に定着しています。

```typescript
// 2026年標準：TypeScript必須
export interface GameEntity {
  id: string;
  position: Vec3;
  velocity: Vec3;
  mesh: THREE.Mesh;
  physics: PhysicsBody;
}

export async function initializeGame(): Promise<GameEngine> {
  const engine = new GameEngine();
  await engine.initialize();
  return engine;
}
```

### 1.4 エンジンの選択肢

#### Three.js
- **利用者数**: 最多
- **特徴**: 自由度が高い、学習リソースが豊富
- **WebGPU対応**: サポート中

```typescript
import * as THREE from 'three';

const renderer = new THREE.WebGPURenderer();
renderer.render(scene, camera);
```

#### Babylon.js
- **ゲーム機能**: Three.jsより充実
- **インターフェース**: Unityに近い
- **WebGPU対応**: ネイティブサポート

```typescript
import * as BABYLON from '@babylonjs/core';

const engine = new BABYLON.Engine(canvas, true);
engine.enableWebGPU = true;
```

#### PlayCanvas
- **開発環境**: ブラウザベース
- **共同開発**: チーム向け機能
- **ホスティング**: インテグレーション完備

#### Bevy（Rust）
- **言語**: 純Rust
- **パフォーマンス**: WASM化でネイティブに近い
- **エコシステム**: Rust開発者から人気急上昇

```rust
// Rustでゲーム開発、WASM化してブラウザ配信
fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_systems(Update, move_player)
        .run();
}
```

### 1.5 Rust + WASM の重要性

ゲームロジックの重い部分はWASM化が一般的です。

**WASM化に適した処理:**

- パスファインディング（A*、Dijkstra）
- 物理演算（剛体、布、粒子）
- AI判定（意思決定、ステートマシン）
- ボーン計算（スキニング、IK）
- 画像処理（リサイズ、フィルター、色空間変換）
- 音声処理（エンコード、スペクトラム解析）
- ネットワーク計算（圧縮、暗号化）

データフロー例:

```text
React UI
  ↓
TypeScript Service
  ↓
Rust WASM
  ├─ Physics Engine
  ├─ AI Logic
  └─ Asset Processing
  ↓
Result → React State → Three.js Render
```

## 2. WebGPU を活用したレンダリング

### 2.1 Three.js + WebGPU の基本セットアップ

```typescript
// src/services/renderer.ts
import * as THREE from 'three';
import WebGPURenderer from 'three/addons/renderers/webgpu/WebGPURenderer.js';

export class GameRenderer {
  private renderer: WebGPURenderer;
  private scene: THREE.Scene;
  private camera: THREE.PerspectiveCamera;

  constructor(canvas: HTMLCanvasElement) {
    this.renderer = new WebGPURenderer({ canvas });
    this.renderer.setPixelRatio(window.devicePixelRatio);
    this.renderer.setSize(window.innerWidth, window.innerHeight);

    this.scene = new THREE.Scene();
    this.camera = new THREE.PerspectiveCamera(
      75,
      window.innerWidth / window.innerHeight,
      0.1,
      1000
    );
    this.camera.position.z = 5;
  }

  async initialize(): Promise<void> {
    // WebGPU初期化は非同期
    await this.renderer.init();
  }

  render(state: GameState): void {
    this.updateScene(state);
    this.renderer.render(this.scene, this.camera);
  }

  private updateScene(state: GameState): void {
    // シーン更新ロジック
  }
}
```

### 2.2 Compute Shader による物理演算

WebGPUのCompute Shaderで大規模な物理計算をGPU上で実行します。

```wgsl
// shaders/particle_compute.wgsl
struct Particle {
  position: vec4f,
  velocity: vec4f,
}

@group(0) @binding(0) var<storage, read_write> particles: array<Particle>;

@compute @workgroup_size(256)
fn update(@builtin(global_invocation_id) global_id: vec3u) {
  let idx = global_id.x;
  if (idx >= arrayLength(&particles)) { return; }

  var p = particles[idx];
  
  // 重力を加える
  p.velocity.y -= 9.8 * 0.016; // 60FPS での delta time
  
  // 位置更新
  p.position += p.velocity * 0.016;
  
  // 境界条件（バウンス）
  if (p.position.y < -10.0) {
    p.position.y = -10.0;
    p.velocity.y = -p.velocity.y * 0.9; // 反発係数
  }
  
  particles[idx] = p;
}
```

Rust + WASMから呼び出す場合:

```rust
// core/src/gpu_physics.rs
#[wasm_bindgen]
pub struct GPUPhysicsEngine {
  device: web_sys::GpuDevice,
  compute_pipeline: web_sys::GpuComputePipeline,
  particle_buffer: web_sys::GpuBuffer,
}

#[wasm_bindgen]
impl GPUPhysicsEngine {
  pub async fn new() -> Result<GPUPhysicsEngine, JsValue> {
    let instance = web_sys::gpu::Gpu::instance();
    let adapter = instance.request_adapter(None).await?;
    let device = adapter.request_device(None).await?;
    
    // パイプライン構築
    // ...
    
    Ok(GPUPhysicsEngine { device, compute_pipeline, particle_buffer })
  }

  pub fn step(&self) {
    // GPU計算を実行
  }
}
```

### 2.3 大量オブジェクト描画（インスタンシング）

```typescript
// src/services/instanced-mesh.ts
export class InstancedGameObjectPool {
  private mesh: THREE.InstancedMesh;
  private dummy = new THREE.Object3D();
  private activeCount = 0;

  constructor(
    geometry: THREE.BufferGeometry,
    material: THREE.Material,
    maxCount: number
  ) {
    this.mesh = new THREE.InstancedMesh(geometry, material, maxCount);
  }

  spawn(position: THREE.Vector3, rotation?: THREE.Euler): number {
    if (this.activeCount >= this.mesh.count) return -1;

    this.dummy.position.copy(position);
    if (rotation) this.dummy.rotation.copy(rotation);
    this.dummy.updateMatrix();

    this.mesh.setMatrixAt(this.activeCount, this.dummy.matrix);
    this.mesh.instanceMatrix.needsUpdate = true;

    return this.activeCount++;
  }

  updateMatrix(index: number, position: THREE.Vector3): void {
    this.dummy.position.copy(position);
    this.dummy.updateMatrix();
    this.mesh.setMatrixAt(index, this.dummy.matrix);
    this.mesh.instanceMatrix.needsUpdate = true;
  }

  despawn(index: number): void {
    // 非表示にする
    this.dummy.position.set(0, -10000, 0);
    this.dummy.updateMatrix();
    this.mesh.setMatrixAt(index, this.dummy.matrix);
    this.mesh.instanceMatrix.needsUpdate = true;
  }

  get scene(): THREE.InstancedMesh {
    return this.mesh;
  }
}
```

### 2.4 パーティクルシステム

```typescript
// src/services/particle-system.ts
export class ParticleSystem {
  private particles: Particle[] = [];
  private instancedMesh: InstancedGameObjectPool;

  constructor(maxParticles: number) {
    const geometry = new THREE.SphereGeometry(0.05, 8, 8);
    const material = new THREE.MeshBasicMaterial({ color: 0xffffff });
    this.instancedMesh = new InstancedGameObjectPool(geometry, material, maxParticles);
  }

  emit(
    position: THREE.Vector3,
    velocity: THREE.Vector3,
    lifetime: number,
    count: number
  ): void {
    for (let i = 0; i < count; i++) {
      const angle = Math.random() * Math.PI * 2;
      const speed = 2 + Math.random() * 3;

      this.particles.push({
        position: position.clone(),
        velocity: new THREE.Vector3(
          Math.cos(angle) * speed,
          Math.random() * 2,
          Math.sin(angle) * speed
        ),
        lifetime,
        maxLifetime: lifetime,
        instanceIndex: this.instancedMesh.spawn(position),
      });
    }
  }

  update(deltaTime: number): void {
    for (let i = this.particles.length - 1; i >= 0; i--) {
      const p = this.particles[i];
      p.lifetime -= deltaTime;

      if (p.lifetime <= 0) {
        this.instancedMesh.despawn(p.instanceIndex);
        this.particles.splice(i, 1);
        continue;
      }

      p.position.addScaledVector(p.velocity, deltaTime);
      p.velocity.y -= 9.8 * deltaTime;

      this.instancedMesh.updateMatrix(p.instanceIndex, p.position);
    }
  }
}

interface Particle {
  position: THREE.Vector3;
  velocity: THREE.Vector3;
  lifetime: number;
  maxLifetime: number;
  instanceIndex: number;
}
```

## 3. ブラウザ内 AI の統合

2026年時点で、AIをブラウザ内で実行することが一般的になっています。これにより、サーバー呼び出しなしでNPC制御、画像解析、音声認識などが可能になります。

### 3.1 CLIP を使った画像認識

CLIP（Contrastive Language-Image Pre-training）は、画像とテキストの意味的な距離を計算できるモデルです。ゲーム開発では以下に活用できます。

- プレイヤーの行動分析
- ゲーム内オブジェクトの意味的分類
- テキストから画像生成（プロンプトベースのゲーム要素）

```typescript
// src/services/clipAnalyzer.ts
import { env, pipeline } from '@xenova/transformers';

env.allowLocalModels = false;

export class CLIPAnalyzer {
  private model: any = null;

  async initialize(): Promise<void> {
    // CLIP モデル をロード
    this.model = await pipeline(
      'zero-shot-image-classification',
      'Xenova/clip-vit-base-patch32'
    );
  }

  async classifyImage(
    imageOrUrl: HTMLImageElement | string,
    labels: string[]
  ): Promise<{ label: string; score: number }[]> {
    if (!this.model) {
      throw new Error('CLIPAnalyzer not initialized');
    }

    const result = await this.model(imageOrUrl, labels);
    
    return result.map((item: any) => ({
      label: item.label,
      score: item.score,
    }));
  }

  async analyzeGameScene(
    canvas: HTMLCanvasElement,
    gameObjects: string[]
  ): Promise<Map<string, number>> {
    const result = await this.classifyImage(canvas, gameObjects);
    const scoreMap = new Map<string, number>();

    for (const item of result) {
      scoreMap.set(item.label, item.score);
    }

    return scoreMap;
  }
}
```

使用例：

```typescript
// ゲーム内で敵がプレイヤーの状態を認識
const clipAnalyzer = new CLIPAnalyzer();
await clipAnalyzer.initialize();

const labels = ['player running', 'player jumping', 'player idle', 'player falling'];
const scores = await clipAnalyzer.analyzeGameScene(gameCanvas, labels);

const dominantState = [...scores.entries()].sort((a, b) => b[1] - a[1])[0][0];
console.log(`Player appears to be: ${dominantState}`);
```

### 3.2 ONNX Runtime Web による推論

ONNX（Open Neural Network Exchange）形式のモデルをWebGPUで高速実行できます。

```typescript
// src/services/aiInference.ts
import * as ort from 'onnxruntime-web';

export class AIInferenceEngine {
  private session: ort.InferenceSession | null = null;

  async initialize(modelPath: string): Promise<void> {
    // GPU アクセラレーションを有効化
    await ort.env.wasm.wasmPaths = '/ort-wasm/';

    this.session = await ort.InferenceSession.create(modelPath, {
      executionProviders: ['webgpu', 'wasm'],
    });
  }

  async predict(inputData: Float32Array, inputShape: number[]): Promise<Float32Array> {
    if (!this.session) {
      throw new Error('AIInferenceEngine not initialized');
    }

    const tensor = new ort.Tensor('float32', inputData, inputShape);
    const feeds = { input: tensor };
    const results = await this.session.run(feeds);

    const output = results[Object.keys(results)[0]];
    return output.data as Float32Array;
  }
}
```

### 3.3 敵AI の実装（ONNX + 意思決定）

```typescript
// src/game/enemy-ai.ts
export class EnemyAI {
  private inferenceEngine: AIInferenceEngine;
  private state: 'idle' | 'chase' | 'attack' | 'retreat' = 'idle';

  constructor(inferenceEngine: AIInferenceEngine) {
    this.inferenceEngine = inferenceEngine;
  }

  async decide(
    enemyState: EnemyState,
    playerState: PlayerState,
    gameState: GameState
  ): Promise<AIAction> {
    // 敵と プレイヤーの状態をベクトル化
    const input = this.vectorizeState(enemyState, playerState, gameState);

    // ニューラルネットワークで行動を決定
    const output = await this.inferenceEngine.predict(input, [1, 20]);

    // 出力の最大値のインデックスから行動を決定
    const actionIndex = this.argmax(Array.from(output));
    return this.indexToAction(actionIndex);
  }

  private vectorizeState(
    enemy: EnemyState,
    player: PlayerState,
    game: GameState
  ): Float32Array {
    const vector = new Float32Array(20);
    vector[0] = enemy.position.x / 100; // 正規化
    vector[1] = enemy.position.y / 100;
    vector[2] = enemy.health / enemy.maxHealth;
    vector[3] = player.position.x / 100;
    vector[4] = player.position.y / 100;
    vector[5] = player.health / player.maxHealth;
    vector[6] = this.distance(enemy.position, player.position) / 50;
    // ... その他の特徴量
    return vector;
  }

  private argmax(arr: number[]): number {
    return arr.indexOf(Math.max(...arr));
  }

  private indexToAction(index: number): AIAction {
    const actions: AIAction[] = [
      { type: 'move', direction: 'up' },
      { type: 'move', direction: 'down' },
      { type: 'move', direction: 'left' },
      { type: 'move', direction: 'right' },
      { type: 'attack' },
      { type: 'defend' },
      { type: 'retreat' },
    ];
    return actions[index % actions.length];
  }

  private distance(a: Vec3, b: Vec3): number {
    return Math.hypot(b.x - a.x, b.y - a.y, b.z - a.z);
  }
}

interface AIAction {
  type: 'move' | 'attack' | 'defend' | 'retreat';
  direction?: 'up' | 'down' | 'left' | 'right';
}
```

### 3.4 リアルタイム音声認識

```typescript
// src/services/voiceCommand.ts
import { pipeline } from '@xenova/transformers';

export class VoiceCommandListener {
  private recognizer: any = null;
  private mediaStream: MediaStream | null = null;

  async initialize(): Promise<void> {
    // 音声認識モデルをロード
    this.recognizer = await pipeline(
      'automatic-speech-recognition',
      'Xenova/whisper-tiny.en'
    );

    // マイクへのアクセスを要求
    this.mediaStream = await navigator.mediaDevices.getUserMedia({ audio: true });
  }

  async recognizeAudio(audioBlob: Blob): Promise<string> {
    if (!this.recognizer) {
      throw new Error('VoiceCommandListener not initialized');
    }

    const result = await this.recognizer(audioBlob);
    return result.text;
  }

  async recordAndRecognize(durationMs: number): Promise<string> {
    if (!this.mediaStream) {
      throw new Error('No media stream available');
    }

    const audioChunks: Blob[] = [];
    const mediaRecorder = new MediaRecorder(this.mediaStream);

    return new Promise((resolve) => {
      mediaRecorder.ondataavailable = (event) => {
        audioChunks.push(event.data);
      };

      mediaRecorder.onstop = async () => {
        const audioBlob = new Blob(audioChunks, { type: 'audio/webm' });
        const text = await this.recognizeAudio(audioBlob);
        resolve(text);
      };

      mediaRecorder.start();
      setTimeout(() => mediaRecorder.stop(), durationMs);
    });
  }
}
```

## 4. 入力処理と状態管理

### 4.1 キーボード入力の管理

```typescript
// src/services/input.ts
export class InputManager {
  private keys: Map<string, boolean> = new Map();

  constructor() {
    window.addEventListener('keydown', (e) => {
      this.keys.set(e.key, true);
    });

    window.addEventListener('keyup', (e) => {
      this.keys.set(e.key, false);
    });
  }

  isKeyPressed(key: string): boolean {
    return this.keys.get(key) ?? false;
  }

  getMovementVector(): { x: number; y: number; z: number } {
    let x = 0;
    let y = 0;
    let z = 0;

    if (this.isKeyPressed('ArrowUp') || this.isKeyPressed('w')) z -= 1;
    if (this.isKeyPressed('ArrowDown') || this.isKeyPressed('s')) z += 1;
    if (this.isKeyPressed('ArrowLeft') || this.isKeyPressed('a')) x -= 1;
    if (this.isKeyPressed('ArrowRight') || this.isKeyPressed('d')) x += 1;
    if (this.isKeyPressed(' ')) y += 1; // ジャンプ

    return { x, y, z };
  }
}
```

### 4.2 ポインター入力（マウス/タッチ）

```typescript
// src/services/pointer.ts
export class PointerManager {
  private position = { x: 0, y: 0 };
  private pressed = false;
  private raycaster = new THREE.Raycaster();
  private mouse = new THREE.Vector2();

  constructor(element: HTMLElement) {
    element.addEventListener('mousemove', (e) => {
      const rect = element.getBoundingClientRect();
      this.mouse.x = ((e.clientX - rect.left) / rect.width) * 2 - 1;
      this.mouse.y = -((e.clientY - rect.top) / rect.height) * 2 + 1;
      this.position = {
        x: e.clientX - rect.left,
        y: e.clientY - rect.top,
      };
    });

    element.addEventListener('mousedown', () => {
      this.pressed = true;
    });

    element.addEventListener('mouseup', () => {
      this.pressed = false;
    });
  }

  getPosition(): { x: number; y: number } {
    return { ...this.position };
  }

  isPressed(): boolean {
    return this.pressed;
  }

  raycast(camera: THREE.Camera, objects: THREE.Object3D[]): THREE.Object3D | null {
    this.raycaster.setFromCamera(this.mouse, camera);
    const intersects = this.raycaster.intersectObjects(objects);
    return intersects.length > 0 ? intersects[0].object : null;
  }
}
```

### 4.3 Zustand を使った状態管理

```typescript
// src/store/gameStore.ts
import { create } from 'zustand';

export interface GameState {
  // プレイヤー状態
  player: {
    position: THREE.Vector3;
    health: number;
    maxHealth: number;
    inventory: GameItem[];
  };

  // ゲーム状態
  score: number;
  level: number;
  isPaused: boolean;
  gameTime: number;

  // エンティティ
  enemies: Enemy[];
  pickups: Pickup[];

  // アクション
  movePlayer: (direction: THREE.Vector3) => void;
  takeDamage: (amount: number) => void;
  addScore: (amount: number) => void;
  updateGameTime: (delta: number) => void;
  spawnEnemy: (enemy: Enemy) => void;
  removeEnemy: (id: string) => void;
  togglePause: () => void;
}

export const useGameStore = create<GameState>((set) => ({
  player: {
    position: new THREE.Vector3(0, 0, 0),
    health: 100,
    maxHealth: 100,
    inventory: [],
  },
  score: 0,
  level: 1,
  isPaused: false,
  gameTime: 0,
  enemies: [],
  pickups: [],

  movePlayer: (direction: THREE.Vector3) => {
    set((state) => ({
      player: {
        ...state.player,
        position: state.player.position.add(direction),
      },
    }));
  },

  takeDamage: (amount: number) => {
    set((state) => ({
      player: {
        ...state.player,
        health: Math.max(0, state.player.health - amount),
      },
    }));
  },

  addScore: (amount: number) => {
    set((state) => ({
      score: state.score + amount,
    }));
  },

  updateGameTime: (delta: number) => {
    set((state) => ({
      gameTime: state.gameTime + delta,
    }));
  },

  spawnEnemy: (enemy: Enemy) => {
    set((state) => ({
      enemies: [...state.enemies, enemy],
    }));
  },

  removeEnemy: (id: string) => {
    set((state) => ({
      enemies: state.enemies.filter((e) => e.id !== id),
    }));
  },

  togglePause: () => {
    set((state) => ({
      isPaused: !state.isPaused,
    }));
  },
}));

interface Enemy {
  id: string;
  position: THREE.Vector3;
  health: number;
  maxHealth: number;
}

interface Pickup {
  id: string;
  position: THREE.Vector3;
  type: string;
}

interface GameItem {
  id: string;
  name: string;
  quantity: number;
}
```

## 5. 物理演算（Rust + WASM）

大規模な物理演算や複雑な衝突判定はRust + WASMで実装し、Three.jsの描画と連携します。

### 5.1 Rust で実装する物理エンジン

```rust
// core/src/physics/mod.rs
use wasm_bindgen::prelude::*;
use serde::{Deserialize, Serialize};

#[derive(Clone, Copy, Serialize, Deserialize)]
pub struct Vec3 {
    pub x: f32,
    pub y: f32,
    pub z: f32,
}

impl Vec3 {
    pub fn new(x: f32, y: f32, z: f32) -> Self {
        Vec3 { x, y, z }
    }

    pub fn magnitude(&self) -> f32 {
        (self.x * self.x + self.y * self.y + self.z * self.z).sqrt()
    }

    pub fn distance_to(&self, other: &Vec3) -> f32 {
        let dx = other.x - self.x;
        let dy = other.y - self.y;
        let dz = other.z - self.z;
        (dx * dx + dy * dy + dz * dz).sqrt()
    }
}

#[wasm_bindgen]
pub struct RigidBody {
    pub id: u32,
    pub position: Vec3,
    pub velocity: Vec3,
    pub acceleration: Vec3,
    pub mass: f32,
    pub radius: f32,
    pub is_dynamic: bool,
}

#[wasm_bindgen]
impl RigidBody {
    #[wasm_bindgen(constructor)]
    pub fn new(id: u32, x: f32, y: f32, z: f32, mass: f32, radius: f32) -> RigidBody {
        RigidBody {
            id,
            position: Vec3::new(x, y, z),
            velocity: Vec3::new(0.0, 0.0, 0.0),
            acceleration: Vec3::new(0.0, -9.8, 0.0),
            mass,
            radius,
            is_dynamic: true,
        }
    }

    pub fn apply_force(&mut self, fx: f32, fy: f32, fz: f32) {
        if !self.is_dynamic {
            return;
        }
        self.acceleration.x += fx / self.mass;
        self.acceleration.y += fy / self.mass;
        self.acceleration.z += fz / self.mass;
    }

    pub fn update(&mut self, delta_time: f32) {
        if !self.is_dynamic {
            return;
        }

        self.velocity.x += self.acceleration.x * delta_time;
        self.velocity.y += self.acceleration.y * delta_time;
        self.velocity.z += self.acceleration.z * delta_time;

        self.position.x += self.velocity.x * delta_time;
        self.position.y += self.velocity.y * delta_time;
        self.position.z += self.velocity.z * delta_time;

        // 加速度をリセット（毎フレーム）
        self.acceleration.x = 0.0;
        self.acceleration.y = -9.8;
        self.acceleration.z = 0.0;
    }
}

#[wasm_bindgen]
pub struct PhysicsWorld {
    bodies: Vec<RigidBody>,
    gravity: f32,
    damping: f32,
}

#[wasm_bindgen]
impl PhysicsWorld {
    #[wasm_bindgen(constructor)]
    pub fn new() -> PhysicsWorld {
        PhysicsWorld {
            bodies: Vec::new(),
            gravity: -9.8,
            damping: 0.99,
        }
    }

    pub fn add_body(&mut self, body: RigidBody) -> u32 {
        self.bodies.push(body);
        (self.bodies.len() - 1) as u32
    }

    pub fn step(&mut self, delta_time: f32) {
        // 物理更新
        for body in &mut self.bodies {
            body.update(delta_time);
            body.velocity.x *= self.damping;
            body.velocity.y *= self.damping;
            body.velocity.z *= self.damping;
        }

        // 衝突検出と解決
        self.resolve_collisions();
    }

    fn resolve_collisions(&mut self) {
        let len = self.bodies.len();
        for i in 0..len {
            for j in i + 1..len {
                let distance = self.bodies[i].position.distance_to(&self.bodies[j].position);
                let min_distance = self.bodies[i].radius + self.bodies[j].radius;

                if distance < min_distance {
                    self.handle_collision(i, j);
                }
            }
        }
    }

    fn handle_collision(&mut self, i: usize, j: usize) {
        let pos_i = self.bodies[i].position;
        let pos_j = self.bodies[j].position;

        let distance = pos_i.distance_to(&pos_j);
        if distance == 0.0 {
            return;
        }

        // 衝突法線
        let nx = (pos_j.x - pos_i.x) / distance;
        let ny = (pos_j.y - pos_i.y) / distance;
        let nz = (pos_j.z - pos_i.z) / distance;

        // 相対速度
        let dvx = self.bodies[j].velocity.x - self.bodies[i].velocity.x;
        let dvy = self.bodies[j].velocity.y - self.bodies[i].velocity.y;
        let dvz = self.bodies[j].velocity.z - self.bodies[i].velocity.z;

        let dot = dvx * nx + dvy * ny + dvz * nz;
        if dot >= 0.0 {
            return; // 離れていく方向
        }

        // 衝突応答（反発係数 = 0.8）
        let restitution = 0.8;
        let impulse = -(1.0 + restitution) * dot / 2.0;

        self.bodies[i].velocity.x -= impulse * nx;
        self.bodies[i].velocity.y -= impulse * ny;
        self.bodies[i].velocity.z -= impulse * nz;

        self.bodies[j].velocity.x += impulse * nx;
        self.bodies[j].velocity.y += impulse * ny;
        self.bodies[j].velocity.z += impulse * nz;
    }

    pub fn get_state_json(&self) -> String {
        let positions: Vec<_> = self.bodies.iter()
            .map(|b| (b.id, b.position))
            .collect();
        serde_json::to_string(&positions).unwrap()
    }
}
```

### 5.2 TypeScript から Rust 物理エンジンを呼び出す

```typescript
// src/services/physicsService.ts
import init, { PhysicsWorld, RigidBody } from '../pkg/core';

export class PhysicsService {
  private world: PhysicsWorld | null = null;

  async initialize(): Promise<void> {
    await init();
    this.world = new PhysicsWorld();
  }

  addBody(id: number, x: number, y: number, z: number, mass: number, radius: number): void {
    if (!this.world) throw new Error('PhysicsWorld not initialized');
    
    const body = new RigidBody(id, x, y, z, mass, radius);
    this.world.add_body(body);
  }

  step(deltaTime: number): void {
    if (!this.world) throw new Error('PhysicsWorld not initialized');
    this.world.step(deltaTime);
  }

  getState(): Array<[number, { x: number; y: number; z: number }]> {
    if (!this.world) throw new Error('PhysicsWorld not initialized');
    const json = this.world.get_state_json();
    return JSON.parse(json);
  }

  applyForce(bodyId: number, fx: number, fy: number, fz: number): void {
    // 注：実装上は body を直接アクセスする必要があるため
    // Rust 側で get_body_mut を追加する必要があります
  }
}
```

### 5.3 ゲームループでの物理統合

```typescript
// src/hooks/useGameLoop.ts
import { useEffect } from 'react';
import { useGameStore } from '../store/gameStore';
import { PhysicsService } from '../services/physicsService';

export function useGameLoop(physicsService: PhysicsService) {
  const { updateGameTime } = useGameStore();

  useEffect(() => {
    let rafId: number;
    let lastTime = performance.now();

    const tick = (currentTime: number) => {
      const deltaTime = Math.min((currentTime - lastTime) / 1000, 0.016); // Max 60 FPS
      lastTime = currentTime;

      // 物理シミュレーション
      physicsService.step(deltaTime);

      // ゲーム状態更新
      updateGameTime(deltaTime);

      rafId = requestAnimationFrame(tick);
    };

    rafId = requestAnimationFrame(tick);

    return () => cancelAnimationFrame(rafId);
  }, [physicsService, updateGameTime]);
}
```

## 6. アセット管理と配信の最適化

2026年時点で、ブラウザゲームのアセット配信には以下の形式が標準化されています。

### 6.1 推奨アセット形式

| 形式 | 用途 | 特徴 |
|------|------|------|
| **glTF/glb** | 3Dモデル | 標準フォーマット、圧縮効率、広いツール対応 |
| **KTX2** | テクスチャ | GPU圧縮、高速ロード、ストレージ削減 |
| **WebP** | テクスチャ | 高圧縮率、PNG/JPGより小さい |
| **Basis Universal** | 圧縮テクスチャ | GPU・CPU両対応、最高圧縮率 |

### 6.2 アセットマネージャー

```typescript
// src/services/assetManager.ts
import * as THREE from 'three';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { KTX2Loader } from 'three/addons/loaders/KTX2Loader.js';

export class AssetManager {
  private textureLoader = new THREE.TextureLoader();
  private glftLoader = new GLTFLoader();
  private ktx2Loader: KTX2Loader;
  
  private loadedModels: Map<string, THREE.Group> = new Map();
  private loadedTextures: Map<string, THREE.Texture> = new Map();
  private loadPromises: Map<string, Promise<any>> = new Map();

  constructor(renderer: THREE.WebGLRenderer | THREE.WebGPURenderer) {
    this.ktx2Loader = new KTX2Loader();
    this.ktx2Loader.setTranscoderPath('./ktx2-transcoder/');
    this.ktx2Loader.detectSupport(renderer);
  }

  async loadModel(key: string, path: string): Promise<THREE.Group> {
    if (this.loadedModels.has(key)) {
      return this.loadedModels.get(key)!;
    }

    if (this.loadPromises.has(key)) {
      return this.loadPromises.get(key)!;
    }

    const promise = new Promise<THREE.Group>((resolve, reject) => {
      this.glftLoader.load(
        path,
        (gltf) => {
          this.loadedModels.set(key, gltf.scene);
          resolve(gltf.scene);
        },
        undefined,
        reject
      );
    });

    this.loadPromises.set(key, promise);
    return promise;
  }

  async loadTexture(key: string, path: string): Promise<THREE.Texture> {
    if (this.loadedTextures.has(key)) {
      return this.loadedTextures.get(key)!;
    }

    if (this.loadPromises.has(key)) {
      return this.loadPromises.get(key)!;
    }

    const promise = new Promise<THREE.Texture>((resolve, reject) => {
      if (path.endsWith('.ktx2')) {
        this.ktx2Loader.load(
          path,
          (texture) => {
            this.loadedTextures.set(key, texture);
            resolve(texture);
          },
          undefined,
          reject
        );
      } else {
        this.textureLoader.load(
          path,
          (texture) => {
            this.loadedTextures.set(key, texture);
            resolve(texture);
          },
          undefined,
          reject
        );
      }
    });

    this.loadPromises.set(key, promise);
    return promise;
  }

  async preload(assets: AssetDefinition[]): Promise<void> {
    const promises = assets.map((asset) => {
      if (asset.type === 'model') {
        return this.loadModel(asset.key, asset.path);
      } else if (asset.type === 'texture') {
        return this.loadTexture(asset.key, asset.path);
      }
      return Promise.resolve();
    });

    await Promise.all(promises);
  }

  dispose(): void {
    this.loadedTextures.forEach((texture) => texture.dispose());
    this.loadedModels.clear();
    this.loadedTextures.clear();
    this.loadPromises.clear();
  }
}

interface AssetDefinition {
  key: string;
  path: string;
  type: 'model' | 'texture';
}
```

### 6.3 ローディング画面

```typescript
// src/components/LoadingScreen.tsx
import React, { useEffect, useState } from 'react';
import { AssetManager } from '../services/assetManager';

interface LoadingScreenProps {
  assetManager: AssetManager;
  assets: AssetDefinition[];
  onComplete: () => void;
}

export function LoadingScreen({ assetManager, assets, onComplete }: LoadingScreenProps) {
  const [progress, setProgress] = useState(0);

  useEffect(() => {
    (async () => {
      try {
        const promises = assets.map((asset, index) => {
          return assetManager
            .preload([asset])
            .then(() => {
              setProgress(((index + 1) / assets.length) * 100);
            });
        });

        await Promise.all(promises);
        onComplete();
      } catch (error) {
        console.error('Failed to load assets:', error);
      }
    })();
  }, [assetManager, assets, onComplete]);

  return (
    <div style={styles.container}>
      <div style={styles.progressBar}>
        <div style={{ ...styles.progress, width: `${progress}%` }} />
      </div>
      <p style={styles.text}>{Math.round(progress)}%</p>
    </div>
  );
}

const styles = {
  container: {
    display: 'flex',
    flexDirection: 'column' as const,
    alignItems: 'center',
    justifyContent: 'center',
    height: '100vh',
    backgroundColor: '#000',
  },
  progressBar: {
    width: '300px',
    height: '30px',
    backgroundColor: '#333',
    borderRadius: '15px',
    overflow: 'hidden',
    marginBottom: '20px',
  },
  progress: {
    height: '100%',
    backgroundColor: '#00ff00',
    transition: 'width 0.3s ease',
  },
  text: {
    color: '#fff',
    fontSize: '18px',
  },
};
```

## 7. パフォーマンス最適化

### 7.1 WebGPU パフォーマンスの最大化

```typescript
// src/services/performanceOptimizer.ts
export class PerformanceOptimizer {
  /**
   * WebGPUでCompute Shaderを使用して大規模な計算を高速化
   */
  static configureComputeShader(device: GPUDevice, shader: string): GPUComputePipeline {
    const shaderModule = device.createShaderModule({ code: shader });
    
    return device.createComputePipeline({
      layout: 'auto',
      compute: { module: shaderModule, entryPoint: 'main' },
    });
  }

  /**
   * インスタンシングでドローコール削減
   */
  static createInstancedGeometry(
    geometry: THREE.BufferGeometry,
    count: number
  ): THREE.BufferGeometry {
    const instanceMatrix = new THREE.InstancedBufferGeometry();
    instanceMatrix.copy(geometry);
    
    const instanceMatrices = new Float32Array(16 * count);
    for (let i = 0; i < count; i++) {
      const offset = i * 16;
      // 単位行列で初期化
      const matrix = new THREE.Matrix4();
      const array = matrix.toArray();
      for (let j = 0; j < 16; j++) {
        instanceMatrices[offset + j] = array[j];
      }
    }

    instanceMatrix.instanceMatrix = new THREE.BufferAttribute(instanceMatrices, 16);
    return instanceMatrix;
  }

  /**
   * テクスチャのミップマッピングと最適化
   */
  static optimizeTexture(texture: THREE.Texture): THREE.Texture {
    texture.generateMipmaps = true;
    texture.minFilter = THREE.LinearMipmapLinearFilter;
    texture.magFilter = THREE.LinearFilter;
    texture.anisotropy = 16; // AnisotropicFilteringを有効化
    return texture;
  }

  /**
   * カメラフラスタムカリング
   */
  static setupFrustumCulling(scene: THREE.Scene, camera: THREE.Camera): void {
    const frustum = new THREE.Frustum();
    frustum.setFromProjectionMatrix(
      new THREE.Matrix4().multiplyMatrices(camera.projectionMatrix, camera.matrixWorldInverse)
    );

    scene.traverse((object) => {
      if (object instanceof THREE.Mesh) {
        const inView = frustum.intersectsObject(object);
        object.visible = inView;
      }
    });
  }

  /**
   * LOD（Level of Detail）の実装
   */
  static setupLOD(mesh: THREE.Mesh, lowPolyMesh: THREE.Mesh, mediumPolyMesh: THREE.Mesh): THREE.LOD {
    const lod = new THREE.LOD();
    lod.addLevel(mesh, 0);
    lod.addLevel(mediumPolyMesh, 50);
    lod.addLevel(lowPolyMesh, 100);
    return lod;
  }
}
```

### 7.2 メモリ管理とキャッシング

```typescript
// src/services/memoryManager.ts
export class MemoryManager {
  private static readonly MAX_CACHED_GEOMETRIES = 100;
  private cachedGeometries: Map<string, THREE.BufferGeometry> = new Map();

  getOrCreateGeometry(key: string, factory: () => THREE.BufferGeometry): THREE.BufferGeometry {
    if (this.cachedGeometries.has(key)) {
      return this.cachedGeometries.get(key)!;
    }

    if (this.cachedGeometries.size >= MemoryManager.MAX_CACHED_GEOMETRIES) {
      // 最も古いエントリを削除
      const oldestKey = this.cachedGeometries.keys().next().value;
      const geo = this.cachedGeometries.get(oldestKey);
      if (geo) geo.dispose();
      this.cachedGeometries.delete(oldestKey);
    }

    const geometry = factory();
    this.cachedGeometries.set(key, geometry);
    return geometry;
  }

  /**
   * 不要なメモリをクリーンアップ
   */
  cleanup(): void {
    this.cachedGeometries.forEach((geo) => geo.dispose());
    this.cachedGeometries.clear();
  }

  /**
   * メモリ使用量をモニタリング
   */
  getMemoryInfo(): { geometries: number; textures: number } {
    return {
      geometries: this.cachedGeometries.size,
      textures: 0, // Three.jsから取得可能
    };
  }
}
```

### 7.3 フレームレート監視とアダプティブレンダリング

```typescript
// src/services/performanceMonitor.ts
export class PerformanceMonitor {
  private frameCount = 0;
  private fps = 60;
  private lastTime = performance.now();
  private frameTimeHistory: number[] = [];
  private readonly HISTORY_SIZE = 60;

  update(): void {
    const now = performance.now();
    const frameTime = now - this.lastTime;
    this.lastTime = now;

    this.frameTimeHistory.push(frameTime);
    if (this.frameTimeHistory.length > this.HISTORY_SIZE) {
      this.frameTimeHistory.shift();
    }

    this.frameCount++;
    if (this.frameCount % 60 === 0) {
      this.fps = Math.round(1000 / (frameTime || 1));
    }
  }

  getFPS(): number {
    return this.fps;
  }

  getAverageFrameTime(): number {
    if (this.frameTimeHistory.length === 0) return 0;
    const sum = this.frameTimeHistory.reduce((a, b) => a + b, 0);
    return sum / this.frameTimeHistory.length;
  }

  /**
   * パフォーマンスに基づいて品質を自動調整
   */
  getAdaptiveQuality(): 'high' | 'medium' | 'low' {
    const avgFrameTime = this.getAverageFrameTime();
    if (avgFrameTime < 16.67) {
      // 60+ FPS
      return 'high';
    } else if (avgFrameTime < 33.33) {
      // 30+ FPS
      return 'medium';
    } else {
      return 'low';
    }
  }

  /**
   * パフォーマンス統計を取得
   */
  getStats(): {
    fps: number;
    avgFrameTime: number;
    minFrameTime: number;
    maxFrameTime: number;
  } {
    if (this.frameTimeHistory.length === 0) {
      return { fps: 0, avgFrameTime: 0, minFrameTime: 0, maxFrameTime: 0 };
    }

    const avg = this.getAverageFrameTime();
    const min = Math.min(...this.frameTimeHistory);
    const max = Math.max(...this.frameTimeHistory);

    return {
      fps: this.fps,
      avgFrameTime: avg,
      minFrameTime: min,
      maxFrameTime: max,
    };
  }
}
```

### 7.4 ダイナミックグレードダウン

```typescript
// src/services/dynamicQuality.ts
export class DynamicQuality {
  private quality: 'high' | 'medium' | 'low' = 'high';
  private renderer: THREE.WebGPURenderer;
  private performanceMonitor: PerformanceMonitor;

  constructor(renderer: THREE.WebGPURenderer, monitor: PerformanceMonitor) {
    this.renderer = renderer;
    this.performanceMonitor = monitor;
  }

  update(): void {
    const adaptiveQuality = this.performanceMonitor.getAdaptiveQuality();

    if (adaptiveQuality !== this.quality) {
      this.applyQuality(adaptiveQuality);
      this.quality = adaptiveQuality;
      console.log(`Quality changed to: ${this.quality}`);
    }
  }

  private applyQuality(quality: 'high' | 'medium' | 'low'): void {
    switch (quality) {
      case 'high':
        this.renderer.setPixelRatio(window.devicePixelRatio);
        // シャドウマップ解像度を高く
        break;
      case 'medium':
        this.renderer.setPixelRatio(1);
        // 中程度の解像度
        break;
      case 'low':
        this.renderer.setPixelRatio(0.5);
        // 低解像度でレンダリング
        break;
    }
  }
}
```

## 8. デバッグと開発支援

### 8.1 ゲームデバッガーUI

```typescript
// src/components/DebugPanel.tsx
import React, { useState } from 'react';
import { PerformanceMonitor } from '../services/performanceMonitor';
import { useGameStore } from '../store/gameStore';

interface DebugPanelProps {
  performanceMonitor: PerformanceMonitor;
}

export function DebugPanel({ performanceMonitor }: DebugPanelProps) {
  const [expanded, setExpanded] = useState(false);
  const gameState = useGameStore();
  const stats = performanceMonitor.getStats();

  if (!expanded) {
    return (
      <button
        onClick={() => setExpanded(true)}
        style={{
          position: 'fixed',
          bottom: '20px',
          right: '20px',
          padding: '10px 20px',
          backgroundColor: '#333',
          color: '#0f0',
          border: 'none',
          cursor: 'pointer',
          fontFamily: 'monospace',
        }}
      >
        [DEBUG]
      </button>
    );
  }

  return (
    <div
      style={{
        position: 'fixed',
        bottom: '20px',
        right: '20px',
        backgroundColor: '#000',
        border: '2px solid #0f0',
        color: '#0f0',
        padding: '15px',
        fontFamily: 'monospace',
        fontSize: '12px',
        maxWidth: '300px',
        maxHeight: '400px',
        overflowY: 'auto',
      }}
    >
      <div style={{ marginBottom: '10px' }}>
        <strong>Performance</strong>
        <div>FPS: {stats.fps}</div>
        <div>Avg Frame: {stats.avgFrameTime.toFixed(2)}ms</div>
        <div>Min: {stats.minFrameTime.toFixed(2)}ms</div>
        <div>Max: {stats.maxFrameTime.toFixed(2)}ms</div>
      </div>

      <div style={{ marginBottom: '10px' }}>
        <strong>Game State</strong>
        <div>Player Health: {gameState.player.health}/{gameState.player.maxHealth}</div>
        <div>Score: {gameState.score}</div>
        <div>Level: {gameState.level}</div>
        <div>Enemies: {gameState.enemies.length}</div>
      </div>

      <div style={{ marginBottom: '10px' }}>
        <strong>Player Position</strong>
        <div>X: {gameState.player.position.x.toFixed(2)}</div>
        <div>Y: {gameState.player.position.y.toFixed(2)}</div>
        <div>Z: {gameState.player.position.z.toFixed(2)}</div>
      </div>

      <button
        onClick={() => setExpanded(false)}
        style={{
          width: '100%',
          padding: '5px',
          backgroundColor: '#0f0',
          color: '#000',
          border: 'none',
          cursor: 'pointer',
        }}
      >
        Close
      </button>
    </div>
  );
}
```

### 8.2 Three.js デバッグヘルパー

```typescript
// src/services/debugHelper.ts
import * as THREE from 'three';

export class DebugHelper {
  /**
   * 物理オブジェクトの衝突球を可視化
   */
  static visualizeCollisionSpheres(
    scene: THREE.Scene,
    bodies: { position: THREE.Vector3; radius: number }[]
  ): void {
    const sphereGeometry = new THREE.SphereGeometry(1, 8, 8);
    const sphereMaterial = new THREE.MeshBasicMaterial({
      color: 0x00ff00,
      wireframe: true,
    });

    bodies.forEach((body, index) => {
      const mesh = new THREE.Mesh(sphereGeometry, sphereMaterial);
      mesh.position.copy(body.position);
      mesh.scale.multiplyScalar(body.radius);
      mesh.name = `collision-sphere-${index}`;
      scene.add(mesh);
    });
  }

  /**
   * カメラフラスタムを可視化
   */
  static visualizeFrustum(scene: THREE.Scene, camera: THREE.Camera): void {
    const frustumHelper = new THREE.CameraHelper(camera as THREE.PerspectiveCamera);
    scene.add(frustumHelper);
  }

  /**
   * ライティング情報を表示
   */
  static visualizeLights(scene: THREE.Scene): void {
    scene.traverse((object) => {
      if (object instanceof THREE.Light) {
        const helper = new THREE.LightHelper(object, 10);
        scene.add(helper);
      }
    });
  }

  /**
   * グリッドを追加
   */
  static addDebugGrid(scene: THREE.Scene, size: number = 100): void {
    const gridHelper = new THREE.GridHelper(size, 10, 0x0000ff, 0x808080);
    scene.add(gridHelper);
  }

  /**
   * 座標軸を追加
   */
  static addDebugAxes(scene: THREE.Scene, size: number = 50): void {
    const axesHelper = new THREE.AxesHelper(size);
    scene.add(axesHelper);
  }
}
```

### 8.3 ネットワークプロファイリング

```typescript
// src/services/networkProfiler.ts
export class NetworkProfiler {
  private requests: Map<string, RequestInfo> = new Map();

  trackRequest(url: string, startTime: number): void {
    this.requests.set(url, {
      url,
      startTime,
      endTime: 0,
      size: 0,
    });
  }

  completeRequest(url: string, endTime: number, size: number): void {
    const request = this.requests.get(url);
    if (request) {
      request.endTime = endTime;
      request.size = size;
    }
  }

  /**
   * ネットワーク統計を取得
   */
  getStats(): {
    totalRequests: number;
    totalSize: number;
    avgTime: number;
  } {
    const requests = Array.from(this.requests.values());
    const completedRequests = requests.filter((r) => r.endTime > 0);

    if (completedRequests.length === 0) {
      return { totalRequests: 0, totalSize: 0, avgTime: 0 };
    }

    const totalSize = completedRequests.reduce((sum, r) => sum + r.size, 0);
    const avgTime =
      completedRequests.reduce((sum, r) => sum + (r.endTime - r.startTime), 0) /
      completedRequests.length;

    return {
      totalRequests: completedRequests.length,
      totalSize,
      avgTime,
    };
  }

  logStats(): void {
    const stats = this.getStats();
    console.log(`Network Stats:
      Requests: ${stats.totalRequests}
      Total Size: ${(stats.totalSize / 1024 / 1024).toFixed(2)}MB
      Avg Time: ${stats.avgTime.toFixed(2)}ms`);
  }
}

interface RequestInfo {
  url: string;
  startTime: number;
  endTime: number;
  size: number;
}
```

## 9. クロスプラットフォーム戦略

### 9.1 Web版とDesktop版（Tauri）の共有コア

Rust Coreをboth バージョンで共有することで、実装の重複を避けます。

```text
Web版 (WASM)
  ↓
Rust Core (Shared Game Logic)
  ↑
Desktop版 (Tauri + Rust)
```

**共有対象:**

- Entity管理
- 物理演算
- AI判定
- ゲームルール
- データシリアライズ

### 9.2 環境固有の処理の分離

```typescript
// src/adapters/platform.ts
export interface PlatformAdapter {
  saveGame(data: GameSaveData): Promise<void>;
  loadGame(): Promise<GameSaveData | null>;
  exportVideo(frames: Blob[]): Promise<Blob>;
  getMaxGraphicsQuality(): 'low' | 'medium' | 'high';
}

export class WebPlatformAdapter implements PlatformAdapter {
  async saveGame(data: GameSaveData): Promise<void> {
    const dataStr = JSON.stringify(data);
    localStorage.setItem('game-save', dataStr);
  }

  async loadGame(): Promise<GameSaveData | null> {
    const data = localStorage.getItem('game-save');
    return data ? JSON.parse(data) : null;
  }

  async exportVideo(frames: Blob[]): Promise<Blob> {
    // ブラウザではビデオ出力不可
    throw new Error('Video export not available in Web version');
  }

  getMaxGraphicsQuality() {
    return 'medium';
  }
}

export class TauriPlatformAdapter implements PlatformAdapter {
  async saveGame(data: GameSaveData): Promise<void> {
    await invoke('save_game', { data });
  }

  async loadGame(): Promise<GameSaveData | null> {
    return invoke('load_game');
  }

  async exportVideo(frames: Blob[]): Promise<Blob> {
    return invoke('export_video', { frames });
  }

  getMaxGraphicsQuality() {
    return 'high';
  }
}

export function getPlatformAdapter(): PlatformAdapter {
  if (typeof window !== 'undefined' && '__TAURI_INTERNALS__' in window) {
    return new TauriPlatformAdapter();
  }
  return new WebPlatformAdapter();
}
```

## 10. セーブ・ロード機能

### 10.1 ゲーム状態のシリアライズ（Rust）

```rust
// core/src/game/save.rs
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Clone)]
pub struct GameSaveData {
    pub player_state: PlayerState,
    pub world_state: WorldState,
    pub level: u32,
    pub score: u32,
    pub playtime_seconds: f32,
    pub timestamp: u64,
}

#[derive(Serialize, Deserialize, Clone)]
pub struct PlayerState {
    pub position: Vec3,
    pub health: u32,
    pub max_health: u32,
    pub inventory: Vec<InventoryItem>,
}

#[derive(Serialize, Deserialize, Clone)]
pub struct InventoryItem {
    pub id: String,
    pub name: String,
    pub quantity: u32,
}

#[wasm_bindgen]
pub struct GameEngine {
    // ...
}

#[wasm_bindgen]
impl GameEngine {
    pub fn create_save_data(&self) -> String {
        let save = GameSaveData {
            player_state: self.player_state.clone(),
            world_state: self.world_state.clone(),
            level: self.current_level,
            score: self.score,
            playtime_seconds: self.total_playtime,
            timestamp: std::time::SystemTime::now()
                .duration_since(std::time::UNIX_EPOCH)
                .unwrap()
                .as_secs(),
        };
        serde_json::to_string(&save).unwrap()
    }

    pub fn load_save_data(&mut self, save_json: &str) -> Result<(), JsValue> {
        let save: GameSaveData = serde_json::from_str(save_json)
            .map_err(|e| JsValue::from_str(&e.to_string()))?;

        self.player_state = save.player_state;
        self.world_state = save.world_state;
        self.current_level = save.level;
        self.score = save.score;
        self.total_playtime = save.playtime_seconds;

        Ok(())
    }
}
```

### 10.2 セーブUIの実装

```typescript
// src/components/SaveLoadUI.tsx
import React from 'react';
import { getPlatformAdapter } from '../adapters/platform';
import { useGameStore } from '../store/gameStore';

export function SaveLoadUI() {
  const platformAdapter = getPlatformAdapter();
  const gameState = useGameStore();

  const handleSave = async () => {
    try {
      const saveData = {
        player_state: gameState.player,
        world_state: { enemies: gameState.enemies, pickups: gameState.pickups },
        level: gameState.level,
        score: gameState.score,
        playtime_seconds: gameState.gameTime,
      };

      await platformAdapter.saveGame(saveData);
      alert('Game saved successfully!');
    } catch (error) {
      console.error('Failed to save game:', error);
      alert('Failed to save game');
    }
  };

  const handleLoad = async () => {
    try {
      const saveData = await platformAdapter.loadGame();
      if (!saveData) {
        alert('No save data found');
        return;
      }

      // ゲーム状態を復元
      console.log('Loading game...');
      // gameStore に saveData を反映
    } catch (error) {
      console.error('Failed to load game:', error);
      alert('Failed to load game');
    }
  };

  return (
    <div style={{ display: 'flex', gap: '10px', padding: '10px' }}>
      <button onClick={handleSave} style={buttonStyle}>
        Save Game
      </button>
      <button onClick={handleLoad} style={buttonStyle}>
        Load Game
      </button>
    </div>
  );
}

const buttonStyle = {
  padding: '10px 20px',
  backgroundColor: '#0066cc',
  color: '#fff',
  border: 'none',
  borderRadius: '5px',
  cursor: 'pointer',
};
```

## 11. 推奨事項（2026年標準）

### パフォーマンス

- **WebGPU を活用**: Compute Shaderで大規模計算を高速化
- **インスタンシング**: 大量オブジェクト描画時に必須
- **GPU圧縮テクスチャ**: KTX2またはBasis Universal形式で配信
- **LOD（Level of Detail）**: 距離に基づいて詳細度を変更
- **カメラフラスタムカリング**: 画面外のオブジェクトをスキップ

### アーキテクチャ

- **Rust Core を環境に依存しない層にする**
- **TypeScriptはUI と入出力処理に専念させる**
- **Zustand で状態を一元管理**
- **Platform Adapter で環境固有処理を分離**
- **WebGPU + Three.js / Babylon.js で描画**

### 開発フロー

- **WASM は計算集約的な処理のみ**
- **AI は CLIP + ONNX Runtime で実装**
- **セーブデータは JSON + Platform Adapter で管理**
- **デバッグパネルでリアルタイムモニタリング**
- **パフォーマンス計測と自動品質調整**

### セキュリティ（Tauri対応時）

- **IPC境界で入力検証を必ず実施**
- **ファイルアクセスはサンドボックス内に限定**
- **APIキーなどの秘密情報はサーバー側に配置**
- **セーブデータは暗号化してから保存**

## 12. 技術選定まとめ

**最先端の構成（2026年標準）:**

```
Frontend
├─ React 19 + TypeScript 5
├─ Zustand (状態管理)
└─ TailwindCSS (UI)

Rendering
├─ Three.js r170+ (WebGPU対応版)
└─ WebGPU (ハードウェアアクセラレーション)

Computing
├─ Rust 1.80+
├─ WASM (wasm-pack)
└─ Rayon (並列処理)

AI
├─ CLIP (画像認識)
├─ ONNX Runtime Web
└─ Transformers.js (言語処理)

Asset
├─ glTF 2.0 (.glb)
├─ KTX2 (テクスチャ圧縮)
└─ WebP (画像)

Build
├─ Vite (フロントエンド)
└─ Cargo (Rust)

Deployment
├─ Web: Static hosting (Vercel, Netlify)
└─ Desktop: Tauri (Windows, macOS, Linux)
```

この構成により、

✅ **商用レベルのパフォーマンス**  
✅ **スケーラブルなアーキテクチャ**  
✅ **開発効率と保守性**  
✅ **クロスプラットフォーム対応**  

が実現できます。

## 13. まとめ

Webブラウザでのゲーム開発は、2026年時点で以下の理由から急速に進化しています。

1. **WebGPU** による GPU アクセラレーション
2. **Rust + WASM** による高性能計算
3. **ブラウザ内 AI** による高度なゲーム機能
4. **インスタンシング / LOD** による大規模描画対応
5. **Platform Adapter** による環境隔離

これらのテクノロジーを適切に組み合わせることで、**デスクトップゲームと遜色ないブラウザゲーム**を開発できます。

特にノードベースエディタと CLIP を組み合わせた開発環境は、**現在の Web ゲーム業界の最先端に近いアーキテクチャ**です。

---

## 14. Webゲームの公開手順

Webゲームの公開は、思っているより簡単です。

**TypeScript + Rust + WebGPU** で開発する場合、現在は次の流れが主流です。

```
開発
  ↓
ビルド
  ↓
GitHubへPush
  ↓
自動デプロイ
  ↓
URL公開
```

### 14.1 リリース用ビルドを作成

例えば Vite の場合：

```bash
npm run build
```

実行後、以下が生成されます：

```
dist/
├─ index.html
├─ assets/
└─ ...
```

この `dist` フォルダがそのまま公開物になります。

### 14.2 公開先を選ぶ

**現在のおすすめ順位：**

#### 🥇 Cloudflare Pages（最もおすすめ）

```
✓ 無料
✓ 高速
✓ 独自ドメイン可
✓ GitHub連携
```

URL例：`https://mygame.pages.dev`

#### 🥈 GitHub Pages（最も簡単）

```
✓ 無料
✓ GitHubのみで完結
✓ セットアップが最小限
```

URL例：`https://username.github.io/mygame/`

#### 🥉 Netlify（初心者向け）

```
✓ 無料枠あり
✓ ドラッグ&ドロップ公開可能
✓ GUIが直感的
```

#### その他：Vercel（React系向け）

```
✓ Next.jsとの相性◎
✓ エッジ関数対応
```

### 14.3 GitHubにアップロード

```bash
git init

git add .

git commit -m "first release"

git push origin main
```

### 14.4 社内GitLabへアップロード

企業内でWebゲームや開発ツールを公開する場合は、GitHub Pagesではなく社内GitLab Pagesを利用するケースも多くあります。

#### GitLab Pagesとは

GitLab Pagesは、GitHub Pagesと同様に静的サイトを公開できる機能です。

たとえば、次の技術で作成した成果物を公開できます。

```text
React
├─ TypeScript
└─ Three.js
```

リリース用ビルドを実行すると、公開用ファイルが生成されます。

```text
npm run build
  ↓
dist/
```

社内GitLabでは、このファイルをリポジトリからCI/CD経由で公開します。

```text
GitLab Repository
  ↓
GitLab CI/CD
  ↓
GitLab Pages
```

URLは、社内環境の設定によって次のような形式になります。

```text
https://pages.company.co.jp/mygame
https://gitlab.company.co.jp/pages/mygame
```

実際のURL形式は、社内GitLab管理者の設定に従ってください。

#### GitHub Pagesとの比較

| 項目 | GitHub Pages | GitLab Pages |
| --- | --- | --- |
| 静的サイト公開 | ○ | ○ |
| 独自ドメイン | ○ | ○ |
| HTTPS | ○ | ○ |
| CI/CD | GitHub Actions | GitLab CI |
| 社内限定公開 | △ | ◎ |
| VPN内公開 | △ | ◎ |

企業内の閲覧権限やVPNとの組み合わせを重視する場合は、GitLab Pagesの方が管理しやすいことが多くあります。

#### Webゲームの公開も可能

React、Three.js、WebGPUで作成したゲームをビルドすると、次のような構成で出力できます。

```text
dist/
├─ index.html
├─ assets/
└─ wasm/
```

これらをGitLab Pagesへ配置すれば、利用者はブラウザからURLへアクセスするだけでゲームを起動できます。

```text
ブラウザ
  ↓
URLへアクセス
  ↓
即プレイ
```

#### Rust + WASMも公開可能

TypeScript、Rust、WebAssemblyの構成も、そのままGitLab Pagesで公開できます。生成された次のファイルは、通常の静的ファイルとして配信されます。

```text
.wasm
.js
.data
```

#### 社内利用のメリット

社内向けゲームやツールは、次の流れで自動公開できます。

```text
社内GitLab
  ↓
自動ビルド
  ↓
Pages公開
  ↓
社員のみ閲覧
```

ノードベースSE作成ツールも、更新をすぐに共有できます。

```text
GitLabへPush
  ↓
CI/CD実行
  ↓
Pages更新
```

#### リポジトリ自体はPrivateでも、Pagesだけ公開できる？

はい、設定によって可能な場合があります。ただし、GitLabのバージョンと管理者設定によって利用できる範囲が異なります。

一般的な公開方法は、次の3パターンです。

##### パターン1: リポジトリ Private + Pages Public

```text
Repository
└─ Private

GitLab Pages
└─ Public
```

利用者はソースコードを閲覧できませんが、公開URLからWebゲームやツールを利用できます。

```text
https://game.company.jp
```

Webゲームを社外へ配布する場合に使われる構成です。

##### パターン2: リポジトリ Private + Pages 社内限定

```text
Repository
└─ Private

GitLab Pages
└─ VPN内のみ
```

社員にはアクセスを許可し、リポジトリのソースコードは非公開にする、社内ツールや社内ゲーム向けの構成です。

```text
社員だけ利用可能
ソースコードは閲覧不可
```

##### パターン3: リポジトリ Private + Pages 認証必須

```text
Repository
└─ Private

GitLab Pages
└─ GitLabログイン必須
```

アクセス時にGitLab認証を要求し、許可された利用者だけがPagesを表示できる構成です。

```text
Sign in with GitLab
```

##### 実際の運用例

SE作成ツール、ノードエディタ、WebGPUアプリでは、次の構成が現実的です。

```text
Source
└─ 社内GitLab Private

Deploy
└─ GitLab Pages Public
```

利用者は公開URLへアクセスするだけで利用できます。一方、次のような情報はPrivateリポジトリ内で管理できます。

- TypeScript / Rustのソースコード
- 学習データ
- 社内ノウハウ

##### 配信されるフロントエンドコードの注意点

フロントエンドWebアプリの場合、ブラウザーへ配信されたJavaScriptは利用者が取得できます。

```text
RepositoryはPrivate
  ↓
ビルド済みJavaScriptをPagesへ配信
  ↓
利用者も配信済みコードを取得可能
```

そのため、次の情報をブラウザー側へ直接埋め込んではいけません。

- APIキー
- 機密ロジック
- 社内情報

これらはBackend API側で保護し、ブラウザーからは必要な処理だけを呼び出す構成にします。

##### Webゲーム向けの推奨構成

```text
GitLab Repository
└─ Private

GitLab CI/CD
└─ Build

GitLab Pages
└─ Public

Backend API
└─ 社内サーバー
```

この構成なら、ソース管理を非公開にしたまま、URLだけを配布してゲームやツールを公開できます。

実際に利用できるかは、社内GitLab管理者がGitLab Pagesの公開範囲をどのように設定しているかに依存します。まず「PrivateプロジェクトからPublic Pagesを公開できるか」を確認してください。

#### 事前確認

社内GitLabでは、管理者に以下の項目を確認してください。

- [ ] GitLab Pagesが有効か
- [ ] PrivateプロジェクトからPublic Pagesを公開できるか
- [ ] HTTPSが有効か
- [ ] WebGPUを利用できる環境・ブラウザを提供しているか
- [ ] Pagesの公開先URL形式
- [ ] 最大アップロードサイズ
- [ ] WASM配信時のMIMEタイプや圧縮設定

特にWebGPUを利用する場合は、HTTPSまたはローカル環境での実行が必要です。PagesのHTTPS設定が有効か確認してください。

社内GitLabのバージョンと実行環境が確定すれば、React + TypeScript + Rust/WASM向けの`.gitlab-ci.yml`を作成できます。

### 14.5 自動デプロイ設定

**Cloudflare Pagesの場合：**

```
Create Project
  ↓
GitHub連携
  ↓
Repository選択
  ↓
Build Command
   npm run build
  ↓
Output Directory
   dist
```

保存するだけです。

以降は `git push` するたびに自動更新されます。

### 14.6 HTTPS必須

**WebGPUを使う場合、HTTPS で公開する必要があります。**

```
✓ https://game.com
✓ https://pages.dev
✗ http://game.com
```

### 14.7 PWA化（推奨）

最近のWebゲームでは **インストール可能** にすることが増えています。

ユーザー側は **ホーム画面に追加** を押すだけで、見た目は普通のアプリになります。

```
ブラウザゲーム + ネイティブアプリ体験
```

PWA対応の例：

```typescript
// public/manifest.json
{
  "name": "MyGame",
  "short_name": "Game",
  "description": "2026年標準のWebゲーム",
  "start_url": "/",
  "display": "fullscreen",
  "background_color": "#000000",
  "theme_color": "#0066ff",
  "icons": [
    {
      "src": "/icon-192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icon-512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
```

```typescript
// src/main.tsx
// Service Worker登録
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js');
}
```

### 14.8 あなたの構成なら

以前話していた構成：

```
React + TypeScript + Rust(WASM) + WebGPU + ONNX Runtime
```

以下がおすすめです。

**全体構成：**

```
Frontend
├─ React
├─ Zustand
└─ TypeScript

Game Engine
├─ Three.js
└─ WebGPU

AI
├─ ONNX Runtime Web
└─ CLIP

Deploy
├─ GitHub
└─ Cloudflare Pages
```

**公開までの流れ：**

```
開発
  ↓
npm run build
  ↓
GitHub Push
  ↓
Cloudflare Pages 自動公開
  ↓
URL共有
```

### 14.9 プロトタイプ公開の価値

個人開発なら、次のような用途に相性が良いです：

**Steam出版前のプロトタイプ公開**
- フィードバック収集
- ゲームバランス調整
- バグ発見

**ノードベースSE作成ツール**
- **URLだけで試してもらえる**
- インストール不要
- 即座にフィードバック取得

**クリエイティブ系プロジェクト**
- 画像生成AIツール
- 動画編集補助
- 音声処理エディタ

**現在の主流：**

> ツール系・クリエイティブ系プロジェクトでは、**まずWeb版を公開してフィードバックを集めるのが業界標準**になっています。

あなたのプロジェクトは、

- ✅ ノードベースエディタ
- ✅ CLIP活用
- ✅ ONNX Runtime統合
- ✅ 画像・音声処理

という要素を持つため、**Web版でのフィードバック収集が最も効率的**です。

---

## 15. 次のステップ

1. **Web版の完成度を上げる** → Cloudflare Pagesで無料公開
2. **ユーザーフィードバック収集** → 改善ポイント把握
3. **Desktop版（Tauri）開発** → Rust Core再利用
4. **AIモデル最適化** → CLIP / ONNX のプロファイリング
5. **商用化検討** → Steamまたは独立配布
