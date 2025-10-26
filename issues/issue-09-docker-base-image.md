# Docker ベースイメージの最終決定

## ステータス
- 進捗: 未着手
- 参照: https://chatgpt.com/c/68fc84fc-e8a4-8323-a90b-929b329b67a3

## 背景
CUDA・PyTorch のバージョン互換性を踏まえたベースイメージの最終決定が必要です。`nvidia/cuda` と `pytorch/pytorch` が候補として挙がっています。

## タスク
- [ ] 候補イメージのタグ選定（CUDA / cuDNN / PyTorch バージョン）
- [ ] ViT-L/14 に必要な依存関係の確認
- [ ] 選定理由とトレードオフの整理
- [ ] Dockerfile への反映計画の作成

## 関連 Issue
- #issue-03-docker-container-design
