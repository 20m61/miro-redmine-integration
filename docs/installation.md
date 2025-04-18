# インストールガイド

このガイドでは、Miro-Redmine Integrationの設定とインストール方法について説明します。

## 前提条件

- Redmine 4.2.x 以上が実行されているサーバー
- Redmine API アクセスキー
- Miroアカウント（Professional以上推奨）
- モダンウェブブラウザ（Chrome、Firefox、Edgeなど最新バージョン）

## インストール手順

### 1. Redmine側の設定

1. Redmineの管理者アカウントでログインします。
2. 「管理」→「設定」→「API」タブを開きます。
3. 「RESTによるWebサービスを有効にする」にチェックを入れて保存します。
4. 「マイアカウント」ページへ移動し、右側の「APIアクセスキー」欄に表示されているキーをコピーします。
   * キーが表示されていない場合は「APIアクセスキーを表示」をクリックして生成します。
   * このAPIキーは後ほど連携設定で使用します。

### 2. Miro側の設定

1. [Miroデベロッパープラットフォーム](https://developers.miro.com/)にアクセスします。
2. Miroアカウントでログインし、「Your apps」を選択します。
3. 「Create new app」ボタンをクリックします。
4. アプリ情報を入力します：
   * App name: Redmine Integration（または任意の名前）
   * OAuth scopes: boards:read, boards:write
   * Redirect URI: （ここではまだ空白で構いません）
5. 「Create app」ボタンをクリックします。
6. 作成されたアプリの詳細ページで「Client ID」と「Client secret」を取得します。
   * これらの情報は連携設定で使用します。

### 3. 統合ツールのインストール

#### 方法A: サーバーへの直接インストール（推奨）

1. このリポジトリをクローンまたはダウンロードします：
   ```
   git clone https://github.com/20m61/miro-redmine-integration.git
   ```
2. プロジェクトディレクトリに移動します：
   ```
   cd miro-redmine-integration
   ```
3. 設定ファイルをコピーして編集します：
   ```
   cp config.sample.json config.json
   ```
4. `config.json` を任意のエディタで開き、以下の項目を編集します：
   * Redmine URL
   * Redmine API キー
   * Miroの Client ID と Client Secret
5. 依存ライブラリをインストールします：
   ```
   npm install
   ```
6. アプリケーションを起動します：
   ```
   npm start
   ```
7. ブラウザで `http://localhost:3000` にアクセスし、セットアップを完了します。

#### 方法B: Dockerを使ったインストール

1. リポジトリに含まれる Dockerfile を使って Docker イメージをビルドします：
   ```
   docker build -t miro-redmine-integration .
   ```
2. 設定ファイルを準備します：
   ```
   cp config.sample.json config.json
   ```
   * `config.json` を編集して、必要な情報を入力します
3. コンテナを起動します：
   ```
   docker run -p 3000:3000 -v $(pwd)/config.json:/app/config.json miro-redmine-integration
   ```
4. ブラウザで `http://localhost:3000` にアクセスします。

## 動作確認方法

インストールが完了したら、以下の手順で動作確認を行います：

1. ブラウザで `http://localhost:3000` にアクセスします。
2. 「Redmine連携テスト」ボタンをクリックし、Redmine APIが正しく動作しているか確認します。
   * 成功すると「Redmine連携成功」と表示されます。
3. 「Miro連携テスト」ボタンをクリックし、Miro APIが正しく動作しているか確認します。
   * 成功すると「Miro連携成功」と表示されます。
4. 両方のテストが成功したら、「Miroボード設定」に進んでください。

## トラブルシューティング

* API連携に失敗する場合：
  * Redmine/Miroの API キーが正しいか確認してください
  * ファイアウォールやプロキシの設定を確認してください
  * サーバーのログを確認してエラーメッセージを確認してください

* 操作方法やその他の問題については、[troubleshooting.md](troubleshooting.md)を参照してください。
