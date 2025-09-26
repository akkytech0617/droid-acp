# Droid ACPアダプター仕様・設計書 (typescript)
## 1. 概要
### 1.1 プロジェクト概要
Factory.aiのDroid AIコーディングエージェントをAgent Client Protocol (ACP)に対応させ、Zedエディタおよび他のACP準拠エディタから利用可能にするアダプターを開発する。

### 1.2 目的

Droid CLIとACP間のプロトコル変換
エディタ統合によるシームレスな開発体験の提供
既存のDroid機能をエディタUIで活用

### 1.3 スコープ
#### 含まれるもの：

- ACPプロトコルの実装
- Droid CLIとの通信インターフェース
- 基本的なファイル操作とコード編集機能
- 認証と権限管理

#### 含まれないもの：

- Droid CLI自体の改修
- エディタ側の実装
- Factory.ai固有のエンタープライズ機能

## 2. アーキテクチャ設計
### 2.1 システム構成図
mermaidgraph TD
    A[Zed Editor] -->|ACP JSON-RPC| B[Droid ACP Adapter]
    B -->|Process Control| C[Droid CLI Process]
    B -->|State Management| D[Session Manager]
    B -->|Permission Control| E[Permission Handler]
    C -->|Factory API| F[Factory.ai Services]
    
    G[MCP Servers] <-->|MCP Protocol| B
    
    subgraph "Adapter Core"
        B
        D
        E
        H[Message Translator]
        I[Stream Parser]
    end

## 2.2 コンポーネント設計
### 2.2.1 メインアダプター (DroidACPAdapter)
```
typescriptinterface DroidACPAdapter {
  // ライフサイクル管理
  start(): Promise<void>
  stop(): Promise<void>
  
  // ACPコネクション
  connection: AgentSideConnection
  
  // コンポーネント
  droidManager: DroidProcessManager
  sessionManager: SessionManager
  permissionHandler: PermissionHandler
  messageTranslator: MessageTranslator
  streamParser: StreamParser
}
```

### 2.2.2 Droidプロセス管理 (DroidProcessManager)
```
typescriptinterface DroidProcessManager {
  // プロセス制御
  spawn(config: DroidConfig): Promise<void>
  kill(): Promise<void>
  restart(): Promise<void>
  
  // 通信
  send(command: string): Promise<void>
  onOutput(callback: (data: string) => void): void
  
  // 状態管理
  isReady(): boolean
  getStatus(): ProcessStatus
}
```

### 2.2.3 メッセージ変換 (MessageTranslator)
```
typescriptinterface MessageTranslator {
  // ACP → Droid
  translatePrompt(prompt: ACPPrompt): DroidCommand
  translateFileOperation(op: FileOperation): DroidCommand
  
  // Droid → ACP
  parseResponse(output: string): ACPResponse
  parseToolCall(output: string): ToolCall
  parseError(output: string): ACPError
}
```

## 3. プロトコル実装仕様
### 3.1 ACPメソッド実装
#### 3.1.1 初期化フロー
```
typescript// initialize メソッド
async function handleInitialize(params: InitializeParams): Promise<InitializeResponse> {
  return {
    protocolVersion: "0.1.0",
    capabilities: {
      prompts: {
        planning: true,
        toolCalls: true,
        references: ["file", "symbol", "web"]
      },
      tools: {
        fileOperations: true,
        terminal: true,
        git: true
      },
      permissions: {
        levels: ["low", "medium", "high"],
        granularity: "per-operation"
      }
    },
    serverInfo: {
      name: "droid-acp-adapter",
      version: "1.0.0"
    }
  }
}
```

#### 3.1.2 認証フロー
```
typescript// authenticate メソッド
async function handleAuthenticate(params: AuthenticateParams): Promise<AuthenticateResponse> {
  // Factory.ai認証の実装
  const token = process.env.FACTORY_API_KEY || params.credentials?.apiKey;
  
  if (!token) {
    // ブラウザベース認証へフォールバック
    return {
      status: "pending",
      authUrl: "https://app.factory.ai/auth/cli",
      message: "Please authenticate in your browser"
    }
  }
  
  // トークン検証
  await validateFactoryToken(token);
  return { status: "authenticated" }
}
```

#### 3.1.3 セッション管理
```
typescript// session/new メソッド
async function handleNewSession(params: NewSessionParams): Promise<Session> {
  const session = {
    id: generateSessionId(),
    createdAt: new Date().toISOString(),
    context: {
      projectPath: params.workingDirectory,
      files: params.openFiles || [],
      activeFile: params.activeFile
    }
  };
  
  // Droid CLIセッション開始
  await droidManager.send(`cd ${params.workingDirectory}`);
  await droidManager.send('droid'); // インタラクティブモード開始
  
  return session;
}
```

### 3.2 メッセージ処理
#### 3.2.1 プロンプト処理
```
typescriptasync function handlePrompt(params: PromptParams): Promise<void> {
  const { messages, session } = params;
  const userMessage = messages[messages.length - 1];
  
  // コンテキスト参照の処理
  const enrichedPrompt = await processReferences(userMessage);
  
  // Droidへ送信
  await droidManager.send(enrichedPrompt);
  
  // レスポンスのストリーミング
  streamParser.on('chunk', (chunk) => {
    connection.notify('session/update', {
      sessionId: session.id,
      type: 'assistant_message_chunk',
      content: chunk
    });
  });
  
  // ツール実行の検出と権限リクエスト
  streamParser.on('tool_call', async (tool) => {
    if (requiresPermission(tool)) {
      const permission = await requestPermission(tool);
      if (!permission.granted) {
        await droidManager.send('cancel');
        return;
      }
    }
    await processTool(tool);
  });
}
```

#### 3.2.2 ファイル操作
```
typescript// ファイル書き込みの実装
async function handleWriteFile(params: WriteFileParams): Promise<void> {
  const { path, content, session } = params;
  
  // 差分生成
  const diff = await generateDiff(path, content);
  
  // 権限確認
  if (params.requiresConfirmation) {
    const permission = await connection.request('permission/request', {
      operation: 'file_write',
      path,
      diff,
      riskLevel: 'medium'
    });
    
    if (!permission.granted) {
      throw new Error('File write permission denied');
    }
  }
  
  // Droid経由でファイル更新
  await droidManager.send(`@edit ${path}`);
  await droidManager.send(content);
  await droidManager.send('@end_edit');
  
  // 通知
  connection.notify('session/update', {
    sessionId: session.id,
    type: 'file_updated',
    path
  });
}
```

### 3.3 ストリーム処理
#### 3.3.1 Droid出力パーサー
```
typescriptclass DroidStreamParser {
  private buffer: string = '';
  private state: ParserState = 'idle';
  
  parse(chunk: string): void {
    this.buffer += chunk;
    
    // 状態に応じた解析
    switch (this.state) {
      case 'idle':
        this.detectOutputType();
        break;
      case 'thinking':
        this.parseThinkingOutput();
        break;
      case 'code_generation':
        this.parseCodeOutput();
        break;
      case 'tool_execution':
        this.parseToolOutput();
        break;
    }
  }
  
  private detectOutputType(): void {
    if (this.buffer.includes('Planning:')) {
      this.state = 'thinking';
      this.emit('plan_start');
    } else if (this.buffer.includes('```')) {
      this.state = 'code_generation';
      this.emit('code_start');
    } else if (this.buffer.includes('Executing:')) {
      this.state = 'tool_execution';
      this.emit('tool_start');
    }
  }
}
```

## 4. 権限管理
### 4.1 リスクレベル定義
```
typescriptenum RiskLevel {
  LOW = 'low',      // ファイル読み取り、情報取得
  MEDIUM = 'medium', // ファイル書き込み、ローカル実行
  HIGH = 'high'     // git push、システム変更、外部API呼び出し
}

const OPERATION_RISK_MAP = {
  'file_read': RiskLevel.LOW,
  'file_write': RiskLevel.MEDIUM,
  'git_commit': RiskLevel.MEDIUM,
  'git_push': RiskLevel.HIGH,
  'terminal_exec': RiskLevel.HIGH,
  'delete_file': RiskLevel.HIGH
};
```
### 4.2 権限リクエストフロー
```
typescriptclass PermissionHandler {
  async requestPermission(operation: Operation): Promise<Permission> {
    const riskLevel = this.assessRisk(operation);
    
    // 環境変数による自動承認設定
    const autoApproveLevel = process.env.DROID_AUTO_APPROVE_LEVEL;
    if (this.canAutoApprove(riskLevel, autoApproveLevel)) {
      return { granted: true, automatic: true };
    }
    
    // エディタへ権限リクエスト
    const response = await this.connection.request('permission/request', {
      operation: operation.type,
      details: operation.details,
      riskLevel,
      options: [
        { id: 'allow', label: 'Allow' },
        { id: 'allow_all', label: 'Allow all similar' },
        { id: 'deny', label: 'Deny' }
      ]
    });
    
    // 権限キャッシュ更新
    if (response.option === 'allow_all') {
      this.cachePermission(operation.type, true);
    }
    
    return response;
  }
}
```

## 5. MCP統合
### 5.1 MCPサーバー接続
```
typescriptinterface MCPIntegration {
  // MCPサーバー管理
  servers: Map<string, MCPServer>;
  
  // ツール実行
  async callTool(server: string, tool: string, args: any): Promise<any>;
  
  // リソース取得
  async getResource(server: string, resource: string): Promise<any>;
}

// 初期化時にMCPサーバー情報を受け取る
async function initializeMCP(params: InitializeParams): void {
  if (params.mcpServers) {
    for (const server of params.mcpServers) {
      await mcpIntegration.connect(server);
    }
  }
}
```

## 6. エラーハンドリング
### 6.1 エラーコード定義
```
typescriptenum ErrorCode {
  // JSON-RPC標準エラー
  PARSE_ERROR = -32700,
  INVALID_REQUEST = -32600,
  METHOD_NOT_FOUND = -32601,
  
  // カスタムエラー
  DROID_NOT_AVAILABLE = -32001,
  AUTHENTICATION_FAILED = -32002,
  PERMISSION_DENIED = -32003,
  SESSION_NOT_FOUND = -32004,
  FACTORY_API_ERROR = -32005
}
```

### 6.2 エラー処理

```
typescriptclass ErrorHandler {
  handleError(error: any): ACPError {
    if (error.code === 'ENOENT') {
      return {
        code: ErrorCode.DROID_NOT_AVAILABLE,
        message: 'Droid CLI not found. Please install it first.',
        data: { installUrl: 'https://docs.factory.ai/cli/install' }
      };
    }
    
    if (error.message?.includes('authentication')) {
      return {
        code: ErrorCode.AUTHENTICATION_FAILED,
        message: 'Factory.ai authentication failed',
        data: { authUrl: 'https://app.factory.ai/auth' }
      };
    }
    
    // デフォルトエラー
    return {
      code: ErrorCode.INTERNAL_ERROR,
      message: error.message || 'Unknown error occurred'
    };
  }
}
```
## 7. 設定とカスタマイズ
### 7.1 設定オプション
```
typescriptinterface DroidACPConfig {
  // Droid CLI設定
  droidPath?: string;           // デフォルト: 'droid'
  workingDirectory?: string;    // デフォルト: process.cwd()
  
  // 認証設定
  apiKey?: string;              // Factory.ai APIキー
  authMethod?: 'token' | 'browser' | 'auto';
  
  // 権限設定
  autoApproveLevel?: RiskLevel; // 自動承認レベル
  requireConfirmation?: boolean; // 確認必須モード
  
  // パフォーマンス設定
  streamBufferSize?: number;    // デフォルト: 1024
  responseTimeout?: number;      // デフォルト: 30000ms
  
  // デバッグ設定
  debug?: boolean;
  logLevel?: 'error' | 'warn' | 'info' | 'debug';
}
```
###7.2 環境変数
```
bash# 認証
FACTORY_API_KEY=your-api-key

# Droidパス
DROID_EXECUTABLE=/usr/local/bin/droid

# 権限管理
DROID_AUTO_APPROVE_LEVEL=medium
DROID_REQUIRE_CONFIRMATION=false

# デバッグ
DROID_ACP_DEBUG=true
DROID_ACP_LOG_LEVEL=debug
```

## 8. 実装計画
### 8.1 フェーズ1: MVP（2-3週間）
目標： 基本的な対話機能の実現

 プロジェクトセットアップ（TypeScript、ビルド設定）
 ACPプロトコル基本実装（initialize、authenticate）
 Droidプロセス管理
 簡単なプロンプト送受信
 基本的なストリーミング

成果物：

動作する最小限のアダプター
Zedでの基本動作確認

### 8.2 フェーズ2: コア機能（2-3週間）
目標： ファイル操作とコード編集機能

 ファイル読み書き実装
 Diff生成と表示
 権限管理システム
 エラーハンドリング強化
 セッション管理

成果物：

ファイル編集可能なアダプター
権限管理機能

### 8.3 フェーズ3: 高度な機能（2-3週間）
目標： プロダクション品質への改善

 MCP統合
 プランニング機能
 ターミナル実行
 Git操作サポート
 パフォーマンス最適化

成果物：

フル機能のアダプター
ドキュメント完成

### 8.4 フェーズ4: 公開準備（1-2週間）
目標： オープンソース公開

 テストスイート作成
 CI/CD設定
 ドキュメント整備
 npmパッケージ公開
 コミュニティフィードバック対応

成果物：

npm公開パッケージ
GitHubリポジトリ

## 9. テスト戦略
9.1 ユニットテスト
```
typescriptdescribe('MessageTranslator', () => {
  it('should translate ACP prompt to Droid command', () => {
    const prompt = { content: 'Fix the bug in main.js' };
    const command = translator.translatePrompt(prompt);
    expect(command).toBe('Fix the bug in main.js');
  });
  
  it('should handle file references', () => {
    const prompt = { 
      content: 'Update @file:main.js',
      references: [{ type: 'file', path: 'main.js' }]
    };
    const command = translator.translatePrompt(prompt);
    expect(command).toContain('@main.js');
  });
});
```

### 9.2 統合テスト
```
typescriptdescribe('Droid ACP Adapter', () => {
  let adapter: DroidACPAdapter;
  
  beforeEach(async () => {
    adapter = new DroidACPAdapter();
    await adapter.start();
  });
  
  it('should complete full conversation flow', async () => {
    // 初期化
    const initResponse = await adapter.handleInitialize({});
    expect(initResponse.capabilities).toBeDefined();
    
    // セッション作成
    const session = await adapter.handleNewSession({});
    expect(session.id).toBeDefined();
    
    // プロンプト送信
    await adapter.handlePrompt({
      session,
      messages: [{ role: 'user', content: 'Hello' }]
    });
    
    // レスポンス確認
    const updates = await waitForUpdates();
    expect(updates).toContainChunk('Hello');
  });
});
```
## 10. 配布とインストール
###10.1 npmパッケージ
```
json{
  "name": "@community/droid-acp-adapter",
  "version": "1.0.0",
  "description": "ACP adapter for Factory.ai Droid",
  "bin": {
    "droid-acp": "./dist/cli.js"
  },
  "dependencies": {
    "@zed-industries/agent-client-protocol": "^0.4.0",
    "zod": "^3.0.0"
  }
}
```
### 10.2 Zed設定例
```
json{
  "agent_servers": {
    "Factory Droid": {
      "command": "npx",
      "args": ["@community/droid-acp-adapter"],
      "env": {
        "FACTORY_API_KEY": "your-key",
        "DROID_AUTO_APPROVE_LEVEL": "medium"
      }
    }
  }
}
```

## 11. リスクと対策
リスク影響度対策Droid CLIの仕様変更高バージョン固定、後方互換性維持Factory.ai API制限中レート制限実装、エラーハンドリングパフォーマンス問題中ストリーム処理最適化、バッファリングセキュリティ脆弱性高権限管理徹底、入力検証

## 12. 成功指標

技術指標：

レスポンス時間 < 100ms（ローカル操作）
ストリーミング遅延 < 50ms
エラー率 < 1%


採用指標：

npmダウンロード数 > 1000/月
GitHubスター > 100
アクティブユーザー > 50



## 13. 今後の拡張可能性

他エディタ対応： VS Code、Neovim、Emacs
追加機能： コード解析、テスト生成、リファクタリング
エンタープライズ機能： チーム共有、監査ログ、コンプライアンス