# Miro-Redmine Integration

MiroスクラムボードとRedmineの連携ツール - チケット情報の視覚化と効率的なプロジェクト管理

## 概要

Miro-Redmine Integrationは、オンライン協業ホワイトボードプラットフォーム「Miro」とプロジェクト管理ウェブアプリケーション「Redmine」を連携させるツールです。このツールを使用することで、Redmineのチケット情報をMiroボード上で視覚的に管理し、効率的なプロジェクト運営が可能になります。

## 主な機能

- **Redmineチケットの視覚化**: Redmineのチケット情報をMiroボード上に表示
- **双方向同期**: MiroボードでのアップデートをRedmineに反映、またはその逆も可能
- **スクラム/カンバンボード**: Miroのビジュアルボードを活用したアジャイル管理
- **チケット状態の視覚的表現**: ステータス、優先度、担当者などを色やラベルで表現
- **ドラッグ＆ドロップ操作**: 直感的な操作でチケットステータスの変更が可能

## 動作環境

- Redmine 4.2.x 以上
- Miroアカウント（Professional以上推奨）
- モダンウェブブラウザ

## インストール方法

詳細なインストール手順は [installation.md](docs/installation.md) を参照してください。

## 使い方

1. Miroボードでの初期設定方法は [board-setup.md](docs/board-setup.md) を参照
2. Redmine連携の設定については [setup.md](docs/setup.md) を参照
3. 基本的な操作フローは [operation-flow.md](docs/operation-flow.md) に記載

## 技術的詳細

- REST API: RedmineおよびMiro APIを使用
- データ同期: ウェブフック（Webhook）またはポーリングによる自動同期
- 設定管理: JSONベースの設定ファイル

スクリプトの詳細については [scripts.md](docs/scripts.md) を参照してください。

## トラブルシューティング

よくある問題と解決策については [troubleshooting.md](docs/troubleshooting.md) を参照してください。

## ライセンス

このプロジェクトは [MIT ライセンス](LICENSE) の下で公開されています。

## 貢献方法

1. このリポジトリをフォーク
2. 機能追加やバグ修正のためのブランチを作成 (`git checkout -b feature/amazing-feature`)
3. 変更をコミット (`git commit -m 'Add some amazing feature'`)
4. ブランチにプッシュ (`git push origin feature/amazing-feature`)
5. プルリクエストを作成

## 連絡先

質問や提案がある場合は、Issueを作成するか、リポジトリ管理者にお問い合わせください。
