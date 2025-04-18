# Redmine連携設定ガイド

このドキュメントでは、Miro-Redmine Integrationツールで使用するRedmine側の設定方法について説明します。

## 事前準備

- Redmine 4.2.x以上が実行されているサーバー
- 管理者権限を持つRedmineアカウント
- Miro-Redmine Integrationツールのインストール（[installation.md](installation.md)参照）

## Redmine API設定

### 1. API機能の有効化

1. Redmineの管理者アカウントでログインします。
2. トップメニューから「管理」をクリックします。
3. 左側のメニューから「設定」をクリックします。
4. 「API」タブを選択します。
5. 「RESTによるWebサービスを有効にする」にチェックを入れます。
6. 「JSONP形式を有効にする」（オプション）にチェックを入れます。
7. 「保存」ボタンをクリックします。

### 2. APIアクセスキーの取得

1. Redmineにログインした状態で、右上のユーザー名をクリックします。
2. 「マイアカウント」をクリックします。
3. 右側の「APIアクセスキー」セクションを確認します。
   * APIキーが表示されている場合はそれをコピーします。
   * 表示されていない場合は「APIアクセスキーを表示」をクリックして生成します。
4. 生成されたAPIキーをコピーしておきます（Miro-Redmine Integration設定で使用します）。

## プロジェクト設定

### 1. 連携対象プロジェクトの設定

1. Redmineにログインした状態で、連携したいプロジェクトを開きます。
2. プロジェクトのID（URLの末尾の数字や識別子）をメモしておきます。
   * 例: `https://your-redmine.com/projects/test-project` の場合、識別子は `test-project`
   * 例: `https://your-redmine.com/projects/5` の場合、IDは `5`
3. プロジェクトの設定を確認します：
   * 「設定」タブを開きます。
   * 「トラッカー」、「ステータス」などが適切に設定されているか確認します。

### 2. チケットステータスの確認

1. RedmineのトップメニューIから「管理」をクリックします。
2. 左側のメニューから「チケットのステータス」をクリックします。
3. 利用可能なステータス一覧と各ステータスのIDを確認します。
   * 標準的なステータスは以下の通りです：
     * 新規（ID: 1）
     * 進行中（ID: 2）
     * 解決済み（ID: 3）
     * フィードバック（ID: 4）
     * 終了（ID: 5）
     * 却下（ID: 6）
4. カスタムステータスがある場合は、それらのIDと名前も確認します。
5. これらのステータスIDとMiroボードのカラムを対応付けるために使用します。

## ウェブフック設定（オプション但し推奨）

リアルタイム同期を実現するために、Redmineウェブフックを設定することを強く推奨します。

### 1. ウェブフックプラグインのインストール

1. Redmineサーバーのプラグインディレクトリに移動します：
   ```
   cd /path/to/redmine/plugins
   ```

2. 以下のいずれかのウェブフックプラグインをインストールします：
   * [redmine_webhook](https://github.com/suer/redmine_webhook)
   * [redmine_webhook_integration](https://github.com/alphanodes/redmine_webhook_integration)

   例：
   ```
   git clone https://github.com/suer/redmine_webhook.git
   ```

3. 必要な依存関係をインストールします：
   ```
   cd /path/to/redmine
   bundle install
   ```

4. データベースマイグレーションを実行します：
   ```
   bundle exec rake redmine:plugins:migrate RAILS_ENV=production
   ```

5. Redmineサービスを再起動します：
   ```
   sudo service apache2 restart  # Apache使用の場合
   # または
   sudo service nginx restart    # Nginx使用の場合
   ```

### 2. ウェブフック設定の追加

1. Redmineにログインし、連携したいプロジェクトを開きます。
2. 「設定」タブをクリックします。
3. 左側のメニューから「ウェブフック」または「Webhooks」をクリックします。
4. 「新しいウェブフック」ボタンをクリックします。
5. 以下の情報を入力します：
   * 名前: 「Miro-Redmine Integration」
   * URL: Miro-Redmine Integrationツールのウェブフックエンドポイント
     * 例: `http://your-server.com:3000/api/webhooks/redmine`
   * シークレットトークン: 任意の文字列（セキュリティ向上のため推奨）
   * プロジェクト: 連携対象のプロジェクトを選択
   * チケットの作成: 有効
   * チケットの更新: 有効
   * チケットの削除: 有効
   * その他、必要に応じてイベントを選択
6. 「作成」ボタンをクリックして保存します。
7. 作成したウェブフックのシークレットトークンを、Miro-Redmine Integrationの設定ファイルに追加します。

## Miro-Redmine Integrationの設定

ウェブフックを設定した後、Miro-Redmine Integrationの設定ファイルを更新します：

1. `config.json`ファイルを開きます：
   ```
   nano /path/to/miro-redmine-integration/config.json
   ```

2. Redmine関連の設定を編集します：
   ```json
   {
     "redmine": {
       "url": "https://your-redmine-instance.com",
       "apiKey": "your-redmine-api-key",
       "projectId": "your-project-id",
       "webhook": {
         "enabled": true,
         "secret": "your-webhook-secret-token"
       }
     },
     // その他の設定...
   }
   ```

3. ファイルを保存して閉じます。

4. サービスを再起動します：
   ```
   cd /path/to/miro-redmine-integration
   npm restart
   # または
   docker-compose restart
   ```

## ステータスマッピングの設定

Miro-Redmine Integrationの設定ファイルで、Redmineのチケットステータスとカラムの対応関係を設定します：

1. `config.json`ファイルを開きます：
   ```
   nano /path/to/miro-redmine-integration/config.json
   ```

2. `statusMapping`セクションを編集します：
   ```json
   {
     // その他の設定...
     "statusMapping": {
       "1": { "column": "Backlog", "x": 0 },
       "2": { "column": "In Progress", "x": 600 },
       "3": { "column": "Review", "x": 900 },
       "4": { "column": "Feedback", "x": 900 },
       "5": { "column": "Done", "x": 1200 },
       "6": { "column": "Archive", "x": 1500 }
     }
   }
   ```

   * 各キーはRedmineステータスのIDに対応します
   * `column`はMiroボード上のカラム名
   * `x`は横方向の座標（ピクセル単位）

3. ファイルを保存して閉じます。

4. サービスを再起動します。

## 動作確認

設定が完了したら、以下の手順で動作を確認します：

1. Miro-Redmine Integration管理画面（`http://your-server.com:3000`）にアクセスします。
2. 「テスト」タブを開きます。
3. 「Redmine接続テスト」ボタンをクリックして、API接続を確認します。
4. 「ウェブフックテスト」ボタンをクリックして、ウェブフック設定を確認します。
5. 両方のテストが成功したら、「初期同期」を実行してチケットをMiroボードに反映させます。

## 追加設定オプション

### Redmine側のカスタムフィールド設定

カスタムフィールドを使って、Miroボードとの連携をより柔軟に設定できます：

1. Redmine管理画面で「カスタムフィールド」を開きます。
2. 「新しいカスタムフィールド」をクリックします。
3. 以下のようなフィールドを作成することを検討します：
   * `miro_position_x`: Miroボード上のX座標（数値フィールド）
   * `miro_position_y`: Miroボード上のY座標（数値フィールド）
   * `miro_card_color`: カードの色（リストフィールド）
   * `miro_priority_visual`: 視覚的優先度表示（リストフィールド）

これらのカスタムフィールドは、Miro-Redmine Integrationの設定ファイルでマッピングする必要があります：

```json
{
  // その他の設定...
  "customFieldMapping": {
    "miro_position_x": 11,  // カスタムフィールドのID
    "miro_position_y": 12,  // カスタムフィールドのID
    "miro_card_color": 13,  // カスタムフィールドのID
    "miro_priority_visual": 14  // カスタムフィールドのID
  }
}
```

### ユーザーアバター連携

Redmineのユーザーアバターをチケットカード上に表示するには、追加設定が必要です：

1. Redmineで「アバター」機能が有効になっていることを確認します：
   * 管理画面の「設定」→「表示」タブ
   * 「アバターを有効にする」にチェック

2. Miro-Redmine Integrationの設定ファイルで、ユーザーアバター機能を有効にします：
   ```json
   {
     // その他の設定...
     "features": {
       "userAvatars": true
     }
   }
   ```

## 高度な設定

### マルチプロジェクト連携

複数のRedmineプロジェクトを1つのMiroボードに連携する場合：

1. `config.json`ファイルの`projects`セクションに複数のプロジェクトを設定します：
   ```json
   {
     // その他の設定...
     "projects": [
       {
         "id": "project1",
         "name": "Project 1",
         "y": 0  // Y座標オフセット
       },
       {
         "id": "project2",
         "name": "Project 2",
         "y": 1000  // Y座標オフセット
       }
     ]
   }
   ```

2. 各プロジェクトの縦方向の位置（Y座標）を指定して、プロジェクト間の重複を避けます。

### ロールベースのカスタマイズ

ユーザーロールによって表示や機能を変更する場合：

1. RedmineのAPIを通じてユーザーロール情報を取得するための設定：
   ```json
   {
     // その他の設定...
     "features": {
       "roleBasedAccess": true
     },
     "roles": {
       "1": { "name": "管理者", "permissions": ["edit", "move", "create", "delete"] },
       "2": { "name": "開発者", "permissions": ["edit", "move", "create"] },
       "3": { "name": "報告者", "permissions": ["edit"] },
       "4": { "name": "閲覧者", "permissions": [] }
     }
   }
   ```

2. Miroボード上での権限をロールに基づいて制限できます。

## トラブルシューティング

設定中に問題が発生した場合は、[troubleshooting.md](troubleshooting.md)を参照してください。
