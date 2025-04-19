# Miro-Redmine Integration

MiroスクラムボードとRedmineの連携ツール - チケット情報の視覚化と効率的なプロジェクト管理

![Miro-Redmine Integration概要](docs/images/miro-redmine-overview.png)

## 🚀 主なメリット

- **作業の可視化**: チームの作業状況をリアルタイムで視覚的に把握
- **二重作業の削減**: MiroとRedmine間の自動同期により情報更新の手間を省略
- **コラボレーション強化**: リモート/分散チームでも同じ情報を共有しながら協業
- **アジャイル開発の促進**: スクラム/カンバンボードとしての直感的な操作性
- **意思決定の迅速化**: 視覚的な情報共有によるボトルネックの早期発見

## 概要

Miro-Redmine Integrationは、オンライン協業ホワイトボードプラットフォーム「Miro」とプロジェクト管理ウェブアプリケーション「Redmine」を連携させるツールです。このツールを使用することで、Redmineのチケット情報をMiroボード上で視覚的に管理し、効率的なプロジェクト運営が可能になります。

## 主な機能

- **Redmineチケットの視覚化**: Redmineのチケット情報をMiroボード上に表示
- **双方向同期**: MiroボードでのアップデートをRedmineに反映、またはその逆も可能
- **スクラム/カンバンボード**: Miroのビジュアルボードを活用したアジャイル管理
- **チケット状態の視覚的表現**: ステータス、優先度、担当者などを色やラベルで表現
- **ドラッグ＆ドロップ操作**: 直感的な操作でチケットステータスの変更が可能

## 動作環境

- **サーバー要件**:
  - Node.js 14.x 以上
  - npm 6.x 以上
  - 2GB以上のRAM
  - 1GB以上の空きディスク容量
- **Redmine**: 4.2.x 以上
- **Miroアカウント**: Professional以上推奨
- **ブラウザ**:
  - Google Chrome 90以上
  - Mozilla Firefox 88以上
  - Microsoft Edge 90以上
  - Safari 14以上

## クイックスタート

Miro-Redmine Integrationを最速で試す手順:

1. **Redmine APIキーの取得**:
   ```
   Redmineにログイン → マイアカウント → APIアクセスキーを表示
   ```

2. **リポジトリのクローンとインストール**:
   ```bash
   git clone https://github.com/20m61/miro-redmine-integration.git
   cd miro-redmine-integration
   cp config.sample.json config.json
   # config.jsonを編集（最低限RedmineのURLとAPIキーを設定）
   npm install
   npm start
   ```

3. **Miroアプリの認証**:
   ```
   ブラウザで http://localhost:3000 にアクセス → 「Miro認証」ボタンをクリック
   ```

詳細なインストール手順は [installation.md](docs/installation.md) を参照してください。

## 使い方

1. Miroボードでの初期設定方法は [board-setup.md](docs/board-setup.md) を参照
2. Redmine連携の設定については [setup.md](docs/setup.md) を参照
3. 基本的な操作フローは [operation-flow.md](docs/operation-flow.md) に記載

## 実際の使用例

![Miro-Redmine使用例](docs/images/miro-redmine-usage-example.png)

上記はスプリント計画ミーティングでMiro-Redmine Integrationを使用している例です。チームはMiroボード上でRedmineチケットをドラッグ＆ドロップしながら優先順位付けをしています。変更は自動的にRedmineに反映されます。

## 技術的詳細

- REST API: RedmineおよびMiro APIを使用
- データ同期: ウェブフック（Webhook）またはポーリングによる自動同期
- 設定管理: JSONベースの設定ファイル

スクリプトの詳細については [scripts.md](docs/scripts.md) を参照してください。

## トラブルシューティング

よくある問題と解決策については [troubleshooting.md](docs/troubleshooting.md) を参照してください。

エラーメッセージから解決策をすぐに見つけるには、[エラーコード一覧](docs/troubleshooting.md#エラーコード一覧)を参照してください。

## バージョン情報

- 現在のバージョン: 1.0.0 (2025年4月)
- 対応Redmineバージョン: 4.2.x - 5.0.x
- 対応Miroプラン: Professional, Business, Enterprise

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
