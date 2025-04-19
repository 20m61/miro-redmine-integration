# インストールガイド

このガイドでは、Miro-Redmine Integrationの設定とインストール方法について説明します。

## インストールチェックリスト

以下のチェックリストに沿って進めることで、インストールが完了します。各ステップのチェックボックスにチェックを入れながら進めてください。

- [ ] 前提条件の確認
- [ ] Redmine側の設定
- [ ] Miro側の設定
- [ ] 統合ツールのインストール
- [ ] 設定ファイルの編集
- [ ] 動作確認

## 前提条件

- [ ] Redmine 4.2.x 以上が実行されているサーバー
  * 最新版（5.0.x）での動作も確認済み
- [ ] Redmine API アクセスキー
- [ ] Miroアカウント（Professional以上推奨）
  * 無料プランでは機能制限があります
- [ ] Node.js 14.x 以上と npm 6.x 以上
- [ ] モダンウェブブラウザ:
  * Google Chrome 90以上
  * Mozilla Firefox 88以上 
  * Microsoft Edge 90以上
  * Safari 14以上

## インストール手順

### 1. Redmine側の設定

- [ ] Redmineの管理者アカウントでログインします。
- [ ] 「管理」→「設定」→「API」タブを開きます。
- [ ] 「RESTによるWebサービスを有効にする」にチェックを入れて保存します。
  
  ![Redmine API設定画面](images/redmine-api-settings.png)
  
- [ ] 「マイアカウント」ページへ移動し、右側の「APIアクセスキー」欄に表示されているキーをコピーします。
   * キーが表示されていない場合は「APIアクセスキーを表示」をクリックして生成します。
   * このAPIキーは後ほど連携設定で使用します。
   * ⚠️ **重要**: APIキーは秘密情報として安全に管理してください。第三者と共有しないでください。

### 2. Miro側の設定

- [ ] [Miroデベロッパープラットフォーム](https://developers.miro.com/)にアクセスします。
- [ ] Miroアカウントでログインし、「Your apps」を選択します。
- [ ] 「Create new app」ボタンをクリックします。
- [ ] アプリ情報を入力します：
   * App name: Redmine Integration（または任意の名前）
   * OAuth scopes: boards:read, boards:write
   * Redirect URI: サーバーのコールバックURL（例: `http://localhost:3000/auth/miro/callback`）
- [ ] 「Create app」ボタンをクリックします。
- [ ] 作成されたアプリの詳細ページで「Client ID」と「Client secret」を取得します。
   * これらの情報は連携設定で使用します。
   * ⚠️ **重要**: Client secretは秘密情報として安全に管理してください。

### 3. 統合ツールのインストール

#### 方法A: サーバーへの直接インストール（推奨）

- [ ] このリポジトリをクローンまたはダウンロードします：
   ```bash
   git clone https://github.com/20m61/miro-redmine-integration.git
   ```
- [ ] プロジェクトディレクトリに移動します：
   ```bash
   cd miro-redmine-integration
   ```
- [ ] 設定ファイルをコピーして編集します：
   ```bash
   cp config.sample.json config.json
   ```
- [ ] `config.json` を任意のエディタで開き、以下の項目を編集します：
   * Redmine URL
   * Redmine API キー
   * Miroの Client ID と Client Secret
- [ ] 依存ライブラリをインストールします：
   ```bash
   npm install
   ```
- [ ] アプリケーションを起動します：
   ```bash
   npm start
   ```
- [ ] ブラウザで `http://localhost:3000` にアクセスし、セットアップを完了します。

#### 方法B: Dockerを使ったインストール

- [ ] リポジトリに含まれる Dockerfile を使って Docker イメージをビルドします：
   ```bash
   docker build -t miro-redmine-integration .
   ```
- [ ] 設定ファイルを準備します：
   ```bash
   cp config.sample.json config.json
   ```
   * `config.json` を編集して、必要な情報を入力します
- [ ] コンテナを起動します：
   ```bash
   docker run -p 3000:3000 -v $(pwd)/config.json:/app/config.json miro-redmine-integration
   ```
- [ ] ブラウザで `http://localhost:3000` にアクセスします。

## 動作確認方法

インストールが完了したら、以下の手順で動作確認を行います：

1. ブラウザで `http://localhost:3000` にアクセスします。

   ![管理画面トップ](images/admin-dashboard.png)

2. 「Redmine連携テスト」ボタンをクリックし、Redmine APIが正しく動作しているか確認します。
   * 成功すると「Redmine連携成功」と緑色のメッセージが表示されます。
   * 失敗すると赤色のエラーメッセージが表示されます。エラーの詳細も表示されるので、内容に応じて設定を修正してください。

   **成功時の表示:**
   ![Redmine連携成功](images/redmine-test-success.png)

   **失敗時の表示（例: APIキーが無効）:**
   ![Redmine連携失敗](images/redmine-test-failure.png)

3. 「Miro連携テスト」ボタンをクリックし、Miro APIが正しく動作しているか確認します。
   * 成功すると「Miro連携成功」と緑色のメッセージが表示されます。
   * Miro認証が必要な場合は、Miroのログイン画面にリダイレクトされます。指示に従ってログインし、アプリケーションへのアクセスを許可してください。

   ![Miro認証画面](images/miro-auth-screen.png)

4. 両方のテストが成功したら、「Miroボード設定」に進んでください。

5. エラーが発生した場合のトラブルシューティング:
   * **「Redmine APIへの接続に失敗しました」**: Redmine URLとAPIキーを確認。サーバーがアクセス可能か確認。
   * **「Miro認証に失敗しました」**: Client IDとClient Secret、リダイレクトURLを確認。
   * **「データベース接続エラー」**: データベース設定（使用している場合）を確認。
   * その他のエラーについては[troubleshooting.md](troubleshooting.md)を参照してください。

## 環境別インストール注意点

### Windows環境での注意点

- Node.jsはWindows用インストーラーを使用してインストールすることをお勧めします。
- Windowsのパスの区切り文字は「\」ですが、設定ファイルでは「/」または「\\\\」（エスケープされたバックスラッシュ）を使用してください。
- Windows環境ではコマンドプロンプトやPowerShellで実行する際、一部のコマンドが異なる場合があります。
  ```powershell
  # PowerShellでの実行例
  $env:NODE_ENV="production"
  npm start
  ```

### Mac/Linux環境での注意点

- 権限の問題が発生した場合は、以下のコマンドで実行権限を付与してください。
  ```bash
  chmod +x scripts/*.sh
  ```
- 3000番ポートが既に使用されている場合は、config.jsonの「server」セクションで別のポートを指定できます。

### クラウド環境での注意点

- **AWS/Heroku等のクラウドサービスでの実行**:
  * 環境変数を使用して設定を行うことができます。
  * 例: `REDMINE_URL`, `REDMINE_API_KEY`, `MIRO_CLIENT_ID`など
  * 詳細は[cloud-deployment.md](cloud-deployment.md)を参照してください。

- **HTTPS対応**:
  * 本番環境では必ずHTTPS通信を使用することをお勧めします。
  * Let's Encryptなどの無料SSLサービスが利用可能です。
  * リバースプロキシ（Nginx/Apache）の使用も検討してください。

## セキュリティに関する注意事項

1. **APIキーとシークレット**:
   * APIキーやClient Secretなどの認証情報はソースコード管理システム（Gitなど）にコミットしないでください。
   * 本番環境では環境変数や安全なシークレット管理サービスの使用を検討してください。

2. **アクセス制限**:
   * 管理インターフェースへのアクセスは信頼できるネットワークからのみ許可することをお勧めします。
   * ファイアウォールやBasic認証などを使用してアクセスを制限してください。

3. **定期的な更新**:
   * セキュリティ上の理由から、定期的に最新バージョンに更新することをお勧めします。
   * `npm audit` コマンドで依存パッケージのセキュリティチェックを行ってください。

## 次のステップ

インストールが完了したら、以下のドキュメントを参照して設定を進めてください：

- [ボード設定ガイド](board-setup.md) - Miroボードの設定方法
- [Redmine連携設定](setup.md) - Redmineとの詳細な連携設定
- [操作フロー](operation-flow.md) - 日常的な使用方法

## トラブルシューティング

インストール中や設定中に問題が発生した場合は、[troubleshooting.md](troubleshooting.md)を参照するか、GitHubのIssueとして問題を報告してください。
