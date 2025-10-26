# AWS リソース設計と IaC 方針

## ステータス
- 進捗: 調査中
- 参照: https://chatgpt.com/c/68fc84fc-e8a4-8323-a90b-929b329b67a3

## 背景
g6e 系インスタンスで利用可能な L40S GPU（VRAM 48GB）1 枚で ViT-L/14 の推論が可能であることを確認済みです。今後は CloudFormation を用いて必要なリソースを IaC 化する必要があります。

## 未完了タスク
- [ ] CloudFormation テンプレートのスコープ確定（VPC、サブネット、セキュリティグループ、IAM ロールなど）
- [ ] EC2 インスタンス構成の詳細化（AMI、ストレージ、ユーザーデータ）
- [ ] S3 アクセス権限や IAM ポリシーの定義
- [ ] テンプレートの検証とレビュー

## 関連 Issue
- #issue-10-iac-detailed-implementation
