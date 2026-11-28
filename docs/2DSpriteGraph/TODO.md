# 2DSpriteGraph - TODO

2DSpriteGraph は、2D スプライト画像を効率的に管理・編集するためのノードベースグラフエディタです。

## 進捗状況

### 進行中 🔄

- [ ] キャラクターエディターの改修
  - [ ] 共通処理は関数化する（ポーズサンプルにも適用）
  - [ ] IP-Adapter weight の設定UI調整
    - [ ] スライダー（0.0～2.0、0.01刻み）の実装
    - [ ] 数値入力フィールドの実装
    - [ ] 通常モード / 上級者モード の切り替え
    - [ ] 部位別weight設定（Face、Hair、LeftSleeve、RightSleeve、Clothes、Weapon）
    - [ ] プリセット設定（一般用途: 1.0、顔重視: 1.2、服装重視: 1.4、完全固定: 1.8）

### 予定 📋

- [ ]  サンプルプロジェクトを作成
  - [ ]  プロジェクト作成画面で「サンプルから作成」を追加
  - [ ]  asset/sample/以下にサンプルプロジェクト作成
- [ ]  ポーズアセット
  - [ ]  サンプルデータとしてAsset/以下にサンプルを用意する
  - [ ]  他のプロジェクトのポーズをインポートできる様にする
    - [ ]  インポートするポーズを選択する
- [ ]  ポーズの頭身
  - [ ]  1.5頭身
  - [ ]  ２頭身
  - [ ]  ３頭身
  - [ ]  ４頭身
  - [ ]  ５頭身
  - [ ]  ６頭身
  - [ ]  ７頭身（標準）
    - [ ]  ポーズエディターで作成するポーズは７頭身で行う
  - [ ]  頭身をキャラクターエディターで選択できる様にする
    - [ ]  UIを分かりやすくする（特に数値が何を設定するとどうなるのか分からない）
- [ ] ポーズ間で違いが無いようにする
  - [ ]  色違いが無いように判定する
  - [ ]  パーツのサイズ違いが無いように判定する
- [ ]  モーション調整
  - [ ]  分かりやすくする
  - [ ]  動きを大きくする
- [ ] https://huggingface.co/

### IP-Adapter スプライト最適化 ✨

specs/IP-Adapter.md で設計を進行中

- [ ] 部位別参照の実装
  - [ ] Face、Hair、LeftSleeve、RightSleeve、Clothes、Weapon の6部位標準化
  - [ ] 参照画像ごとの weight 設定（Face: 1.0、Clothes: 1.3～1.6、Weapon: 0.7～1.2）
- [ ] 袖マスク自動抽出
  - [ ] DWPose骨格 + セグメンテーション による袖領域の抽出
  - [ ] LeftSleeve / RightSleeve の独立部位化
  - [ ] 長袖→半袖、色変化などのブレを抑制
- [ ] 袖の一貫性を最優先するモード
  - [ ] Clothes Weight: 1.4～1.8
  - [ ] 後処理による袖補正（骨格追従スリーブ方式）

### 複数頭身システムの設計 ✨

specs/複数の頭身のスケルトンを作成してモーションを共有.md で検討中

- [ ] モーションと体型比率の分離設計
  - [ ] Normalized Skeleton の導入
  - [ ] Body Style の頭身別パラメータ化
  - [ ] モーション共有による管理効率化
- [ ] 頭身別の体型比率テーブル
  - [ ] 2頭身：head_ratio 1.0、torso_ratio 0.4、leg_ratio 0.6
  - [ ] 3頭身：head_ratio 0.9、torso_ratio 0.5、leg_ratio 0.9
  - [ ] 6頭身：head_ratio 0.5、torso_ratio 1.0、leg_ratio 1.5
  - [ ] 7頭身（標準）：head_ratio 0.4、torso_ratio 1.2、leg_ratio 2.0
- [ ] 実行時の頭身変換パイプライン
  - [ ] Motion → Normalized Skeleton → Body Style 適用 → DWPose PNG 生成

### マルチ参照IP-Adapter の実装 ✨

specs/Multi-Reference-IP-Adapter.md で設計中

- [ ] Reference Pack Builder ノードの実装
  - [ ] 複数参照画像の重み正規化
  - [ ] 段階的なIP-Adapter適用
- [ ] スプライト用途の最適重み配置
  - [ ] Face: 0.15（顔特徴の維持）
  - [ ] Hair: 0.20（髪型の維持）
  - [ ] Clothes: 0.55（衣装の最優先）
  - [ ] Weapon: 0.10（武器の補助）
- [ ] 袖固定モードのデフォルト設定
  - [ ] LeftSleeve / RightSleeve の独立構成
  - [ ] DWPoseと組み合わせたポーズ制御


### Ideas💡

- [ ] アイデア

### 完了 ✅

- [x] 腰の位置で中央X座標の補正（位置が取得できない場合は、モーションのノード
- [x] サイドねニューのノードは、ノードグラフエディターだけに表示する

## 開発リンク

- [2DSpriteGraph](./index.md)
- [IP-Adapter ガイド](./specs/IP-Adapter.md)
- [Multi Reference IP-Adapter ガイド](./specs/Multi-Reference-IP-Adapter.md)
- [複数頭身のスケルトン共有](./specs/複数の頭身のスケルトンを作成してモーションを共有.md)
- [2頭身でポーズをつける](./specs/2頭身でポーズをつける.md)
