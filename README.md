# 3d-yonnmoku-v2  
4×4×4 立体四目並べ AlphaZero + 詰み検出アルゴリズム

---

## 📖 Overview
本リポジトリは **4×4×4 立体四目並べ（Yonmoku）** を対象とした  
AlphaZero-style 自己対戦学習システムです。  

- **Policy-Value MCTS** による自己対局でデータ生成  
- **Residual CNN** を用いた方策・価値ネットワークの学習  
- **詰み検出 (`solve_mate.py`)** を組み込むことで  
  - 学習の安定化  
  - プレイ中の読みを深める  
- GPU / CPU いずれでも動作（TensorFlow 2 系）

---

## 🗂️ Directory

src/
├── game.py # 盤面クラス・合法手生成・勝敗判定
├── dual_network.py # Keras モデル定義・保存・読み込み
├── pv_mcts.py # Policy-Value MCTS 本体
├── solve_mate.py # 詰み検出アルゴリズム
├── self_play.py # 自己対局データ生成
├── train_net.py # ネットワーク単体学習
├── evaluate_net.py # ネットワーク評価
├── evaluate_best.py # ベストモデルの定期評価
├── train_cycle.py # Self-Play → Train → Eval のループ
└── play_best.py # 人間 vs AI（CLI）
data/ # 自己対局履歴 (.pickle) 等
model/ # 学習済み重み (.h5)


> **NOTE:** 空ディレクトリは `.gitkeep` で保持しています。

---

## 🚀 Quick Start

### 1. 環境構築
```bash
git clone https://github.com/taruto00/3d-yonnmoku-v2.git
cd 3d-yonnmoku-v2
python -m venv .venv && source .venv/bin/activate  # 推奨
pip install -r requirements.txt                   # TensorFlow 2.x など
自己対局データ生成
python src/self_play.py --games 500 --model model/best.h5
ネットワーク学習
python src/train_net.py --data_dir data --epochs 30
評価 & モデル更新
python src/evaluate_best.py --new model/latest.h5 --old model/best.h5
一連の流れを自動実行したい場合は train_cycle.py をお使いください。
--cycles 10 などでサイクル回数を指定できます。

人間と対局
python src/play_best.py

特徴
| 機能             | 概要                                         |
| -------------- | ------------------------------------------ |
| 詰み検出           | `solve_mate.py` が最大 *32 手先* までの必勝／必敗を瞬時に判定 |
| データ拡張          | 自己対局データを 90° 回転対称で 8 倍化                    |
| Early-Stopping | 検証損失が改善しない場合に学習を打ち切り                       |
| GPU 対応         | TensorFlow の GPU ビルドを利用可                   |
| 省メモリ           | ビットボード表現で盤面メモリを削減                          |

| パラメータ           | 値     | 備考                  |
| --------------- | ----- | ------------------- |
| `BOARD_SIZE`    | 4×4×4 |                     |
| `MCTS_SIM`      | 50   | 評価試行数               |
| `CPUCT`         | 1.0  | UCB 定数              |
| `BATCH_SIZE`    | 128   | 学習                  |
| `RES_BLOCKS`    | 12     | Residual Block 数    |
| `FILTERS`       | 96   | Conv フィルタ数          |
| `SP_GAME_COUNT` | 500   | Self-Play ゲーム数/サイクル |


