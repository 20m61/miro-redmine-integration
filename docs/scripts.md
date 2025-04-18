# スクリプト技術仕様

このドキュメントでは、Miro-Redmine Integrationで使用されているスクリプトとそのAPI連携の技術的な詳細について説明します。開発者やカスタマイズを行いたい方向けの情報です。

## アーキテクチャ概要

Miro-Redmine Integrationは以下のコンポーネントで構成されています：

1. **バックエンドサーバー**: Node.js + Expressで実装されたRESTful API
2. **フロントエンドUI**: React/Vue.jsで実装されたWeb管理インターフェース
3. **同期サービス**: ウェブフックと定期的なポーリングを使用したデータ同期
4. **認証/認可モジュール**: OAuth 2.0を使用したAPI認証

各コンポーネント間は非同期通信を基本とし、設定ファイルやデータベースを介して情報を共有します。

## コア機能モジュール

### 1. Redmine API クライアント

```javascript
// redmine-client.js
const axios = require('axios');

class RedmineClient {
  constructor(baseUrl, apiKey) {
    this.baseUrl = baseUrl;
    this.apiKey = apiKey;
    this.client = axios.create({
      baseURL: baseUrl,
      headers: {
        'X-Redmine-API-Key': apiKey,
        'Content-Type': 'application/json'
      }
    });
  }

  // プロジェクト情報の取得
  async getProject(projectId) {
    try {
      const response = await this.client.get(`/projects/${projectId}.json`);
      return response.data.project;
    } catch (error) {
      console.error('Error fetching project:', error);
      throw error;
    }
  }

  // チケット一覧の取得
  async getIssues(projectId, params = {}) {
    try {
      const response = await this.client.get('/issues.json', {
        params: {
          project_id: projectId,
          status_id: 'open',
          limit: 100,
          ...params
        }
      });
      return response.data.issues;
    } catch (error) {
      console.error('Error fetching issues:', error);
      throw error;
    }
  }

  // チケットの詳細取得
  async getIssue(issueId) {
    try {
      const response = await this.client.get(`/issues/${issueId}.json`);
      return response.data.issue;
    } catch (error) {
      console.error('Error fetching issue:', error);
      throw error;
    }
  }

  // チケットの更新
  async updateIssue(issueId, issueData) {
    try {
      const response = await this.client.put(`/issues/${issueId}.json`, {
        issue: issueData
      });
      return response.data;
    } catch (error) {
      console.error('Error updating issue:', error);
      throw error;
    }
  }

  // その他の必要なメソッド...
}

module.exports = RedmineClient;
```

### 2. Miro API クライアント

```javascript
// miro-client.js
const axios = require('axios');

class MiroClient {
  constructor(clientId, clientSecret, redirectUri) {
    this.clientId = clientId;
    this.clientSecret = clientSecret;
    this.redirectUri = redirectUri;
    this.baseUrl = 'https://api.miro.com/v2';
    this.client = axios.create({
      baseURL: this.baseUrl,
      headers: {
        'Content-Type': 'application/json'
      }
    });
  }

  // アクセストークンの設定
  setAccessToken(token) {
    this.client.defaults.headers.common['Authorization'] = `Bearer ${token}`;
  }

  // OAuth認証用URLの生成
  getAuthUrl() {
    return `https://miro.com/oauth/authorize?response_type=code&client_id=${this.clientId}&redirect_uri=${encodeURIComponent(this.redirectUri)}`;
  }

  // アクセストークンの取得
  async getAccessToken(code) {
    try {
      const response = await axios.post('https://api.miro.com/v1/oauth/token', {
        grant_type: 'authorization_code',
        client_id: this.clientId,
        client_secret: this.clientSecret,
        redirect_uri: this.redirectUri,
        code: code
      });
      return response.data;
    } catch (error) {
      console.error('Error getting access token:', error);
      throw error;
    }
  }

  // ボード情報の取得
  async getBoard(boardId) {
    try {
      const response = await this.client.get(`/boards/${boardId}`);
      return response.data;
    } catch (error) {
      console.error('Error fetching board:', error);
      throw error;
    }
  }

  // ボード内のすべてのアイテム取得
  async getBoardItems(boardId, type = null) {
    try {
      let url = `/boards/${boardId}/items`;
      if (type) {
        url += `?type=${type}`;
      }
      const response = await this.client.get(url);
      return response.data;
    } catch (error) {
      console.error('Error fetching board items:', error);
      throw error;
    }
  }

  // スティッキーノートの作成
  async createStickyNote(boardId, data) {
    try {
      const response = await this.client.post(`/boards/${boardId}/sticky_notes`, data);
      return response.data;
    } catch (error) {
      console.error('Error creating sticky note:', error);
      throw error;
    }
  }

  // カード/アイテムの更新
  async updateItem(boardId, itemId, data) {
    try {
      const response = await this.client.patch(`/boards/${boardId}/items/${itemId}`, data);
      return response.data;
    } catch (error) {
      console.error('Error updating item:', error);
      throw error;
    }
  }

  // その他の必要なメソッド...
}

module.exports = MiroClient;
```

### 3. 同期マネージャー

```javascript
// sync-manager.js
const RedmineClient = require('./redmine-client');
const MiroClient = require('./miro-client');

class SyncManager {
  constructor(config) {
    this.redmineClient = new RedmineClient(config.redmine.url, config.redmine.apiKey);
    this.miroClient = new MiroClient(
      config.miro.clientId,
      config.miro.clientSecret,
      config.miro.redirectUri
    );
    this.miroClient.setAccessToken(config.miro.accessToken);
    
    this.boardId = config.miro.boardId;
    this.projectId = config.redmine.projectId;
    this.statusMapping = config.statusMapping || {};
    this.issueCardMap = new Map(); // RedmineチケットIDとMiroカードIDのマッピング
  }

  // 初期同期: Redmineからチケットを取得してMiroボードに反映
  async initialSync() {
    try {
      // Redmineからチケットを取得
      const issues = await this.redmineClient.getIssues(this.projectId);
      
      // チケットをMiroボードに追加
      for (const issue of issues) {
        await this.createCardForIssue(issue);
      }
      
      console.log(`Synced ${issues.length} issues to Miro board`);
      return true;
    } catch (error) {
      console.error('Error during initial sync:', error);
      throw error;
    }
  }

  // Redmineチケットに対応するMiroカードを作成
  async createCardForIssue(issue) {
    // チケット情報からカードデータを構築
    const cardData = this.buildCardDataFromIssue(issue);
    
    // Miroボードにカードを作成
    const card = await this.miroClient.createStickyNote(this.boardId, cardData);
    
    // マッピングを保存
    this.issueCardMap.set(issue.id.toString(), card.id);
    
    return card;
  }

  // チケット情報からカードデータを構築するヘルパーメソッド
  buildCardDataFromIssue(issue) {
    // チケットのステータスに基づいて配置位置やスタイルを決定
    const position = this.getPositionForStatus(issue.status.id);
    const style = this.getStyleForIssue(issue);
    
    return {
      content: `<p><strong>#${issue.id} ${issue.subject}</strong></p>
                <p>担当: ${issue.assigned_to ? issue.assigned_to.name : '未割当'}</p>
                <p>優先度: ${issue.priority.name}</p>`,
      style: style,
      x: position.x,
      y: position.y,
      width: 200,
      metadata: {
        redmine_issue_id: issue.id.toString()
      }
    };
  }

  // その他の必要なメソッド...
}

module.exports = SyncManager;
```

## ウェブフック処理

### Redmineからのウェブフック

```javascript
// redmine-webhook-handler.js
const express = require('express');
const router = express.Router();
const SyncManager = require('./sync-manager');

// 設定の読み込み
const config = require('../config.json');
const syncManager = new SyncManager(config);

// Redmineからのウェブフックを処理
router.post('/redmine', async (req, res) => {
  try {
    const payload = req.body;
    
    // ウェブフックのタイプを確認
    switch (payload.action) {
      case 'issue_created':
        await handleIssueCreated(payload.issue);
        break;
      case 'issue_updated':
        await handleIssueUpdated(payload.issue);
        break;
      case 'issue_deleted':
        await handleIssueDeleted(payload.issue);
        break;
      default:
        console.log(`Unhandled webhook action: ${payload.action}`);
    }
    
    res.status(200).send('Webhook processed');
  } catch (error) {
    console.error('Error processing Redmine webhook:', error);
    res.status(500).send('Error processing webhook');
  }
});

// 新規チケット作成の処理
async function handleIssueCreated(issue) {
  try {
    // 完全なチケット情報を取得
    const fullIssue = await syncManager.redmineClient.getIssue(issue.id);
    
    // Miroボードにカードを作成
    await syncManager.createCardForIssue(fullIssue);
    
    console.log(`Created card for issue #${issue.id} on Miro board`);
  } catch (error) {
    console.error(`Error handling issue created webhook: ${error}`);
    throw error;
  }
}

// チケット更新の処理
async function handleIssueUpdated(issue) {
  // 実装...
}

// チケット削除の処理
async function handleIssueDeleted(issue) {
  // 実装...
}

module.exports = router;
```

### MiroからのWebhook

```javascript
// miro-webhook-handler.js
const express = require('express');
const router = express.Router();
const crypto = require('crypto');
const SyncManager = require('./sync-manager');

// 設定の読み込み
const config = require('../config.json');
const syncManager = new SyncManager(config);

// Miroからのウェブフックを処理
router.post('/miro', async (req, res) => {
  try {
    // Webhookシグネチャの検証
    if (!verifySignature(req)) {
      return res.status(401).send('Invalid signature');
    }
    
    const payload = req.body;
    
    // イベントタイプの確認
    switch (payload.type) {
      case 'item.update':
        await handleItemUpdated(payload.data);
        break;
      case 'item.create':
        await handleItemCreated(payload.data);
        break;
      case 'item.delete':
        await handleItemDeleted(payload.data);
        break;
      default:
        console.log(`Unhandled webhook type: ${payload.type}`);
    }
    
    res.status(200).send('Webhook processed');
  } catch (error) {
    console.error('Error processing Miro webhook:', error);
    res.status(500).send('Error processing webhook');
  }
});

// Webhookシグネチャの検証
function verifySignature(req) {
  const signature = req.headers['x-miro-signature'];
  if (!signature) return false;
  
  const hmac = crypto.createHmac('sha256', config.miro.webhookSecret);
  hmac.update(JSON.stringify(req.body));
  const calculatedSignature = hmac.digest('hex');
  
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(calculatedSignature)
  );
}

// アイテム更新の処理
async function handleItemUpdated(itemData) {
  // 実装...
}

// その他の関数...

module.exports = router;
```

## 設定ファイル

Miro-Redmine Integration は JSON 形式の設定ファイルを使用します：

```javascript
// config.sample.json
{
  "redmine": {
    "url": "https://your-redmine-instance.com",
    "apiKey": "your-redmine-api-key",
    "projectId": "your-project-id",
    "webhook": {
      "enabled": true,
      "secret": "your-webhook-secret"
    }
  },
  "miro": {
    "clientId": "your-miro-client-id",
    "clientSecret": "your-miro-client-secret",
    "redirectUri": "http://localhost:3000/auth/miro/callback",
    "boardId": "your-miro-board-id",
    "accessToken": "your-miro-access-token",
    "webhookSecret": "your-miro-webhook-secret"
  },
  "statusMapping": {
    "1": { "column": "Backlog", "x": 0 },
    "2": { "column": "ToDo", "x": 300 },
    "3": { "column": "In Progress", "x": 600 },
    "4": { "column": "Review", "x": 900 },
    "5": { "column": "Done", "x": 1200 }
  },
  "sync": {
    "interval": 300, // 5分ごとの同期（秒単位）
    "autoSync": true
  },
  "server": {
    "port": 3000,
    "host": "localhost"
  }
}
```

## フロントエンドコンポーネント

フロントエンドUIは、React/Vue.jsフレームワークを使用して構築されています。以下は主要コンポーネントの例です：

### ダッシュボード画面

```javascript
// Dashboard.jsx (React)
import React, { useState, useEffect } from 'react';
import axios from 'axios';

function Dashboard() {
  const [settings, setSettings] = useState(null);
  const [syncStatus, setSyncStatus] = useState('idle');
  const [lastSyncTime, setLastSyncTime] = useState(null);
  
  useEffect(() => {
    // 設定を読み込む
    loadSettings();
    // 同期ステータスを読み込む
    loadSyncStatus();
  }, []);
  
  const loadSettings = async () => {
    try {
      const response = await axios.get('/api/settings');
      setSettings(response.data);
    } catch (error) {
      console.error('Error loading settings', error);
    }
  };
  
  const loadSyncStatus = async () => {
    try {
      const response = await axios.get('/api/sync/status');
      setSyncStatus(response.data.status);
      setLastSyncTime(response.data.lastSyncTime);
    } catch (error) {
      console.error('Error loading sync status', error);
    }
  };
  
  const handleStartSync = async () => {
    try {
      setSyncStatus('syncing');
      await axios.post('/api/sync/start');
      await loadSyncStatus();
    } catch (error) {
      console.error('Error starting sync', error);
      setSyncStatus('error');
    }
  };
  
  if (!settings) {
    return <div>Loading...</div>;
  }
  
  return (
    <div className="dashboard">
      <h1>Miro-Redmine Integration Dashboard</h1>
      
      <div className="status-card">
        <h2>Status</h2>
        <p>Current Status: {syncStatus}</p>
        {lastSyncTime && (
          <p>Last Synced: {new Date(lastSyncTime).toLocaleString()}</p>
        )}
        <button onClick={handleStartSync} disabled={syncStatus === 'syncing'}>
          {syncStatus === 'syncing' ? 'Syncing...' : 'Start Sync'}
        </button>
      </div>
      
      <div className="settings-card">
        <h2>Settings</h2>
        <div className="settings-group">
          <h3>Redmine</h3>
          <p>URL: {settings.redmine.url}</p>
          <p>Project: {settings.redmine.projectId}</p>
        </div>
        
        <div className="settings-group">
          <h3>Miro</h3>
          <p>Board ID: {settings.miro.boardId}</p>
        </div>
      </div>
    </div>
  );
}

export default Dashboard;
```

## コマンドラインインターフェース（CLI）

管理者向けに簡単なCLIコマンドも提供しています：

```javascript
// cli.js
#!/usr/bin/env node
const program = require('commander');
const SyncManager = require('./lib/sync-manager');
const config = require('./config.json');

program
  .version('1.0.0')
  .description('Miro-Redmine Integration CLI');

// 同期コマンド
program
  .command('sync')
  .description('Synchronize Redmine issues to Miro board')
  .action(async () => {
    try {
      console.log('Starting synchronization...');
      const syncManager = new SyncManager(config);
      await syncManager.initialSync();
      console.log('Synchronization completed successfully.');
      process.exit(0);
    } catch (error) {
      console.error('Error during synchronization:', error);
      process.exit(1);
    }
  });

// 設定テストコマンド
program
  .command('test-connection')
  .description('Test connection to Redmine and Miro APIs')
  .action(async () => {
    try {
      console.log('Testing Redmine connection...');
      const syncManager = new SyncManager(config);
      
      // Redmineテスト
      const project = await syncManager.redmineClient.getProject(config.redmine.projectId);
      console.log(`Redmine connection successful. Project: ${project.name}`);
      
      // Miroテスト
      const board = await syncManager.miroClient.getBoard(config.miro.boardId);
      console.log(`Miro connection successful. Board: ${board.name}`);
      
      process.exit(0);
    } catch (error) {
      console.error('Connection test failed:', error);
      process.exit(1);
    }
  });

program.parse(process.argv);
```

## テストスイート

プロジェクトには、Jest/Mochaを使用した単体テストも含まれています。テストは `tests` ディレクトリに格納されています。

```javascript
// tests/redmine-client.test.js
const RedmineClient = require('../lib/redmine-client');
const axios = require('axios');
const MockAdapter = require('axios-mock-adapter');

describe('RedmineClient', () => {
  let client;
  let mock;
  
  beforeEach(() => {
    client = new RedmineClient('https://example.com', 'test-api-key');
    mock = new MockAdapter(axios);
  });
  
  afterEach(() => {
    mock.restore();
  });
  
  test('getProject should return project data', async () => {
    const mockProject = {
      project: {
        id: 1,
        name: 'Test Project',
        description: 'This is a test project'
      }
    };
    
    mock.onGet('https://example.com/projects/1.json').reply(200, mockProject);
    
    const result = await client.getProject(1);
    expect(result).toEqual(mockProject.project);
  });
  
  // 他のテスト...
});
```

## 開発のためのヒント

### APIレート制限への対応

MiroとRedmineのAPIには、レート制限があります。多数のアイテムを同期する場合は、以下の点に注意してください：

1. **バッチ処理**: 多数のアイテムをバッチに分けて処理し、各バッチ間に遅延を入れる
2. **エクスポネンシャルバックオフ**: APIエラーが発生した場合、再試行するまでの待機時間を徐々に増やす
3. **キャッシュ**: 頻繁に変更されないデータはキャッシュし、APIコールを減らす

### 認証フロー

OAuth 2.0フローは以下のステップで実装します：

1. Miroの認証ページにユーザーをリダイレクト
2. 認証コードを取得
3. 認証コードを使ってアクセストークンを取得
4. アクセストークンを設定ファイルまたはデータベースに保存

### データベースサポート

大規模なデプロイメントの場合、ファイルベースの設定の代わりにデータベースを使用できます：

```javascript
// db-manager.js
const mongoose = require('mongoose');

// スキーマ定義
const SettingsSchema = new mongoose.Schema({
  redmine: {
    url: String,
    apiKey: String,
    projectId: String
  },
  miro: {
    clientId: String,
    clientSecret: String,
    boardId: String,
    accessToken: String
  },
  statusMapping: mongoose.Schema.Types.Mixed,
  syncInterval: Number
});

const MappingSchema = new mongoose.Schema({
  redmineIssueId: String,
  miroItemId: String,
  lastSynced: Date
});

// モデル
const Settings = mongoose.model('Settings', SettingsSchema);
const Mapping = mongoose.model('Mapping', MappingSchema);

class DatabaseManager {
  // データベース接続
  static async connect(connectionString) {
    await mongoose.connect(connectionString, {
      useNewUrlParser: true,
      useUnifiedTopology: true
    });
  }
  
  // 設定の取得
  static async getSettings() {
    let settings = await Settings.findOne();
    if (!settings) {
      // デフォルト設定
      settings = new Settings({
        redmine: { url: '', apiKey: '', projectId: '' },
        miro: { clientId: '', clientSecret: '', boardId: '', accessToken: '' },
        statusMapping: {},
        syncInterval: 300
      });
      await settings.save();
    }
    return settings;
  }
  
  // マッピングの取得・更新
  static async getMapping(redmineIssueId) {
    return await Mapping.findOne({ redmineIssueId });
  }
  
  static async updateMapping(redmineIssueId, miroItemId) {
    return await Mapping.findOneAndUpdate(
      { redmineIssueId },
      { redmineIssueId, miroItemId, lastSynced: new Date() },
      { upsert: true, new: true }
    );
  }
}

module.exports = DatabaseManager;
```

## デプロイメント

### Docker Compose 設定例

```yaml
# docker-compose.yml
version: '3'
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    volumes:
      - ./config.json:/app/config.json
    restart: unless-stopped
    depends_on:
      - mongodb
    environment:
      - NODE_ENV=production
      - MONGODB_URI=mongodb://mongodb:27017/miro-redmine
  
  mongodb:
    image: mongo:4.4
    volumes:
      - mongodb_data:/data/db
    restart: unless-stopped
    ports:
      - "27017:27017"

volumes:
  mongodb_data:
```
