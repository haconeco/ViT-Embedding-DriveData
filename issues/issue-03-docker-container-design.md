# Docker コンテナ設計の最終化

## ステータス
- 進捗: 調査済み（未決定）
- 参照: https://chatgpt.com/c/68fc84fc-e8a4-8323-a90b-929b329b67a3

## 背景
CUDA 対応のベースイメージ候補として `nvidia/cuda` および `pytorch/pytorch` を検討しましたが、最終的なベースイメージの決定と Dockerfile 設計が未完了です。

## 未完了タスク
- [ ] ViT-L/14 推論に必要な PyTorch / CUDA バージョンの要件整理
- [ ] ベースイメージ候補の比較（サイズ、ドライバ互換性、メンテナンス状況）
- [ ] 最終ベースイメージの決定
- [ ] 必要パッケージを含む Dockerfile の設計

## 関連 Issue
- #issue-09-docker-base-image
