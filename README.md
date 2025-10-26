# ViT-Embedding-DriveData

自動運転用データセット（nuScenes）を ViT で埋め込み（Embedding）する実験プロジェクト。

---

## 概要

* 目的: nuScenes の公式データ（S3）を対象に **Vision Transformer (ViT)** で画像特徴量を抽出。
* 実行基盤: **AWS EC2 g6e 系（NVIDIA L40S 1GPU）**。
* 実行形態: Docker コンテナ上で推論のみ（再学習なし）。
* ベクトルDB: **Faiss** を採用。
* 可視化: **UMAP** により 2D に次元削減し Notebook で描画。
* データアクセス: まずは **s3fs マウント**でシンプルに読み込み。

---

## アーキテクチャ

```text
AWS S3 (nuScenes 公式)
  ↓ s3fs マウント
EC2 g6e.xlarge (L40S 48GB VRAM ×1)
  ├─ Docker (PyTorch + open-clip + faiss)
  ├─ Embedding 抽出 (ViT-L/14, pool=mean 既定)
  ├─ Faiss インデックス作成/保存
  └─ Jupyter Notebook (UMAP 2D 可視化)
```

---

## 採用モデル

| 項目     | 設定                             |
| ------ | ------------------------------ |
| モデル    | **ViT-L/14 (OpenCLIP 係)**      |
| 入力解像度  | 224×224                        |
| パッチ    | 14×14                          |
| プーリング  | 既定: **mean**（可変: `cls`/`mean`） |
| 学習コーパス | ImageNet-21k / LAION 系         |
| 用途     | 推論のみ（再学習なし）                    |
| ライセンス  | Apache-2.0                     |

> 注: nuScenes 専用にファインチューニング済の公開 ViT チェックポイントは未確認。一般向けモデルを採用。

---

## 実行環境

* **EC2**: g6e.xlarge（L40S×1, 48GB VRAM / 4 vCPU / 32 GiB RAM）
* **OS**: Ubuntu 22.04
* **Docker ベース候補**

  * `pytorch/pytorch:2.2.2-cuda12.1-cudnn8-devel`
  * `nvidia/cuda:12.1.1-cudnn8-devel-ubuntu22.04`
* **主要パッケージ**: `torch`, `torchvision`, `open-clip-torch`, `faiss-gpu`, `umap-learn`, `s3fs`, `boto3`

---

## データ

* **入力**: nuScenes 公式データ（S3 配置を想定）。
* **アクセス**: `s3fs` でマウントして読み込み（高速要件は現時点では不要）。
* **前処理**: 入力解像度はモデルに合わせる（例: Resize→CenterCrop / Pad 等、推奨手順）。

---

## 使い方（ハイレベル）

1. **インフラ**: CloudFormation で VPC / SG / IAM / EC2（g6e）作成。
2. **OS 準備**: NVIDIA ドライバ + Container Toolkit, Docker 起動。
3. **コンテナ**: 既定イメージ起動（または Dockerfile でビルド）。
4. **データ**: `s3fs` で nuScenes をマウント。
5. **Embedding**: ViT-L/14 で特徴量抽出（pool 方式は CLI 引数で可変）。
6. **格納**: Faiss でインデックス化・保存（EBS/Ephemeral）。
7. **可視化**: Notebook で UMAP 2D に投影し描画。

> CloudFormation / スクリプトは `cloudformation/` と `scripts/` 以下に配置予定。

---

## Embedding パイプライン（想定 CLI）

```bash
# 例) プーリング方式を切替可能（mean/cls）
python scripts/embed_vit.py \
  --model vit_l_14 \
  --pretrained openclip \
  --image-dir /mnt/nuscenes/samples/CAM_FRONT \
  --image-size 224 \
  --pool mean \
  --batch-size 8 \
  --out vectors/embeddings.parquet

# Faiss インデックス作成
python scripts/build_faiss.py \
  --embeddings vectors/embeddings.parquet \
  --index-out indexes/nuscenes_vitl14.faiss \
  --type flat   # 他: ivfpq, hnsw 等
```

---

## 出力/保存

* **Embedding**: `.npy` / `.parquet`
* **Faiss**: `.faiss`（+ 必要に応じてメタデータ）
* **バックアップ**: S3 へ保存（インデックス/メタ）

---

## 可視化（UMAP）

* Notebook: `notebooks/umap_visualize.ipynb`
* 手順: Embedding 読込 → UMAP(2D) → 散布図（Matplotlib/Plotly）

---

## ディレクトリ構成（予定）

```text
ViT-Embedding-DriveData/
├─ cloudformation/      # AWS IaC (YAML)
├─ docker/              # Dockerfile / 起動スクリプト
├─ scripts/             # embed_vit.py / build_faiss.py / etc.
├─ notebooks/           # UMAP 可視化ノート
└─ README.md
```

---

## タスク進捗

### ✅ 完了

* 要件整理（nuScenes/S3, g6e, Docker, Faiss, ViT 方針）
* ViT モデル調査 → **ViT-L/14** 採用
* g6e/L40S 仕様確認（1GPU で推論可）

### 🚧 未完/実装中

* nuScenes 解像度と前処理設計（Resize/Crop/Pad）
* Docker ベース最終決定 & Dockerfile 整備
* CloudFormation（VPC/SG/IAM/EC2/S3）
* Embedding/格納スクリプト（ViT/ Faiss）
* UMAP 可視化ノート作成
* スケーラビリティ/コスト設計

### ➕ 追加タスク

* nuScenes 元画像 vs ViT 入力解像度の整合性確認
* g6e インスタンスタイプ検討（xlarge/2xlarge 等）
* Faiss 方式選定（flat / ivfpq / hnsw）

---

## ライセンス

* 本プロジェクト: **Apache-2.0**
* 参照モデル/実装（OpenCLIP/Google ViT 等）: **Apache-2.0**
