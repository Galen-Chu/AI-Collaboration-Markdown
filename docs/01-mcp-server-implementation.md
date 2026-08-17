# MCP Server 實作指南

> 本文同時涵蓋 **TypeScript** 與 **Python** 兩種實作路徑，並串聯
> [AI-Agent-Skill](https://github.com/Galen-Chu/AI-Agent-Skill) 的實際架構。

## 📚 目錄
- [什麼是 MCP Server？](#什麼是-mcp-server)
- [為什麼需要 MCP？](#為什麼需要-mcp)
- [架構設計](#架構設計)
- [TypeScript 實作](#typescript-實作)
- [Python 實作](#python-實作)
- [我的生態系整合](#我的生態系整合)
- [完整範例](#完整範例)
- [最佳實踐](#最佳實踐)

---

## 什麼是 MCP Server？

**MCP (Model Context Protocol)** 是一個開放協議，讓 AI 模型能夠：
- 📁 **直接存取本地檔案系統**
- 🔌 **整合外部工具與 API**
- 🔄 **實現雙向數據流**

### 傳統 vs MCP 對比

| 傳統方式 | MCP 方式 |
|---------|---------|
| 需要手動複製程式碼到對話框 | AI 直接讀取專案檔案 |
| 無法執行外部工具 | 可呼叫系統指令、API |
| Context 限制大 | 動態載入相關內容 |

---

## 為什麼需要 MCP？

### 問題場景 1：大型專案理解
❌ **傳統方式：** 需要將多個檔案內容手動貼上到對話框
✅ **MCP 方式：** AI 直接掃描整個專案結構，自動分析關聯性

### 問題場景 2：即時資料查詢
❌ **傳統方式：** 截止目前的知識，無法取得即時資訊
✅ **MCP 方式：** 透過 API 查詢即時資料庫、API 回應

### 問題場景 3：工具整合
❌ **傳統方式：** 只能提供程式碼建議，無法實際執行
✅ **MCP 方式：** 可直接執行 Git 指令、執行測試、部署流程

### 問題場景 4：與個人知識庫整合（2026 現況）
❌ **傳統方式：** AI 不知道你的筆記、日誌、學習進度
✅ **MCP 方式：** 透過 Obsidian MCP + Notion MCP，AI 直接讀寫你的 vault 和登記簿

---

## Python 實作

### 環境準備

```bash
pip install mcp
```

### 基本 Python MCP Server

```python
"""
Python MCP Server — 專案分析工具
使用官方 mcp 套件（Python SDK）
"""
from mcp.server import Server
from mcp.types import Tool, Resource, TextContent
import json
import os

app = Server("project-analyzer")

@app.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="scan_project",
            description="掃描專案目錄結構",
            inputSchema={
                "type": "object",
                "properties": {
                    "root_path": {"type": "string", "description": "專案根目錄路徑"}
                },
                "required": ["root_path"]
            }
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "scan_project":
        root = arguments["root_path"]
        result = {}
        for entry in os.scandir(root):
            if entry.is_dir() and not entry.name.startswith('.'):
                result[entry.name] = "directory"
            elif entry.is_file():
                result[entry.name] = "file"
        return [TextContent(type="text", text=json.dumps(result, indent=2))]

if __name__ == "__main__":
    from mcp.server.stdio import stdio_server
    import asyncio
    asyncio.run(stdio_server(app))
```

### 在 Claude Desktop 中使用（Python）

```json
{
  "mcpServers": {
    "project-analyzer": {
      "command": "python",
      "args": ["path/to/server.py"]
    }
  }
}
```

---

## 我的生態系整合

我在 [AI-Agent-Skill](https://github.com/Galen-Chu/AI-Agent-Skill) 中使用了兩個 MCP Server：

| MCP Server | 用途 | 服務對象 |
|-----------|------|---------|----------|
| **Notion MCP** | Agent 執行狀態追蹤（登記簿讀寫） | `mod-registry-sync` 模組 |
| **Obsidian MCP** | 內容寫入（日記、Work Log） | `personal-assistant` |

### 為什麼選擇這個分工？

```
                    ┌─────────────────┐
                    │  Notion MCP     │  ← 狀態追蹤（跨雲端/本機）
                    │  (登記簿)       │     wf-news-digest 等雲端 pipeline 用
                    └─────────────────┘
                    ┌─────────────────┐
                    │  Obsidian MCP   │  ← 內容典藏（僅本機）
                    │  (vault 寫入)    │     personal-assistant 本機觸發用
                    └─────────────────┘
```

**關鍵限制**：Obsidian MCP 依賴本機 127.0.0.1 REST API，需要 Obsidian 應用程式保持執行中。因此雲端 pipeline（Claude.ai 排程）一律走 Notion，只有本機觸發的才走 Obsidian。

### 實際應用場景

| Pipeline | 使用哪個 MCP | 為什麼 |
|---------|-------------|--------|
| `wf-news-digest` | Notion | Claude.ai 排程，連不到本機 |
| `personal-assistant` | Obsidian | 本機觸發（cron），可直連 vault |

---

## 架構設計

### MCP 核心概念

```
┌─────────────────────────────────────────────────────────┐
│                      AI 模型層                           │
│                    (Claude, GPT-4)                       │
└──────────────────────┬──────────────────────────────────┘
                       │ Model Context Protocol
                       ▼
┌─────────────────────────────────────────────────────────┐
│                     MCP Server                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │  Tools   │  │ Resources│  │ Prompts  │              │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘              │
└───────┼─────────────┼─────────────┼─────────────────────┘
        │             │             │
        ▼             ▼             ▼
   ┌─────────┐  ┌─────────┐  ┌─────────┐
   │ 檔案系統 │  │  資料庫  │  │外部 API │
   └─────────┘  └─────────┘  └─────────┘
```

### MCP Server 三大核心能力

1. **Tools (工具呼叫)**
   - 讓 AI 能執行特定操作
   - 範例：執行測試、查詢資料庫、發送 HTTP 請求

2. **Resources (資源存取)**
   - 提供檔案、資料庫等資源給 AI
   - 範例：讀取檔案內容、查詢設定檔

3. **Prompts (提示範本)**
   - 預定義的提示範本，簡化操作
   - 範例：「幫我審查這個 PR」、「產生單元測試」

---

## 實作步驟

### 步驟 1：環境準備

```bash
# 建立 MCP Server 專案
mkdir my-mcp-server
cd my-mcp-server

# 初始化專案
npm init -y

# 安裝 MCP SDK
npm install @modelcontextprotocol/sdk
```

### 步驟 2：基本 MCP Server 結構

```typescript
// src/index.ts
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';

// 建立 Server 實例
const server = new Server(
  {
    name: 'my-mcp-server',
    version: '1.0.0'
  },
  {
    capabilities: {
      tools: {},      // 啟用工具功能
      resources: {},  // 啟用資源功能
      prompts: {}     // 啟用提示功能
    }
  }
);

// 啟動 Server
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error('MCP Server running on stdio');
}

main().catch(console.error);
```

### 步驟 3：添加 Tools (工具)

```typescript
// 註冊工具 - 檔案讀取
server.setRequestHandler(
  CallToolRequestSchema,
  async (request) => {
    const { name, arguments: args } = request.params;

    if (name === 'read_file') {
      const filePath = args.path;
      try {
        const content = fs.readFileSync(filePath, 'utf-8');
        return {
          content: [{
            type: 'text',
            text: content
          }]
        };
      } catch (error) {
        return {
          content: [{
            type: 'text',
            text: `Error: ${error.message}`
          }],
          isError: true
        };
      }
    }

    // 更多工具...
  }
);

// 列出可用工具
server.setRequestHandler(
  ListToolsRequestSchema,
  async () => {
    return {
      tools: [
        {
          name: 'read_file',
          description: '讀取檔案內容',
          inputSchema: {
            type: 'object',
            properties: {
              path: {
                type: 'string',
                description: '檔案路徑'
              }
            },
            required: ['path']
          }
        }
      ]
    };
  }
);
```

### 步驟 4：添加 Resources (資源)

```typescript
// 註冊資源
server.setRequestHandler(
  ListResourcesRequestSchema,
  async () => {
    return {
      resources: [
        {
          uri: 'file:///config.json',
          name: '專案設定檔',
          description: '目前專案的設定檔內容',
          mimeType: 'application/json'
        }
      ]
    };
  }
);

server.setRequestHandler(
  ReadResourceRequestSchema,
  async (request) => {
    const uri = request.params.uri;
    if (uri === 'file:///config.json') {
      const config = JSON.parse(
        fs.readFileSync('./config.json', 'utf-8')
      );
      return {
        contents: [{
          uri,
          mimeType: 'application/json',
          text: JSON.stringify(config, null, 2)
        }]
      };
    }
  }
);
```

### 步驟 5：添加 Prompts (提示範本)

```typescript
// 註冊提示範本
server.setRequestHandler(
  ListPromptsRequestSchema,
  async () => {
    return {
      prompts: [
        {
          name: 'code_review',
          description: '審查程式碼品質',
          arguments: [
            {
              name: 'file_path',
              description: '要審查的檔案路徑',
              required: true
            }
          ]
        }
      ]
    };
  }
);

server.setRequestHandler(
  GetPromptRequestSchema,
  async (request) => {
    const { name, arguments: args } = request.params;

    if (name === 'code_review') {
      const filePath = args.file_path;
      const code = fs.readFileSync(filePath, 'utf-8');

      return {
        messages: [
          {
            role: 'user',
            content: {
              type: 'text',
              text: `請審查以下程式碼的品質，並提供改進建議：\n\n${code}`
            }
          }
        ]
      };
    }
  }
);
```

---

## 完整範例

### 案例：專案分析 MCP Server

這個 MCP Server 可以：
1. 📊 掃描專案結構
2. 🔍 分析程式碼依賴
3. 📝 產生專案文件

```typescript
// src/project-analyzer-server.ts
import fs from 'fs/promises';
import path from 'path';
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from '@modelcontextprotocol/sdk/types.js';

const server = new Server(
  {
    name: 'project-analyzer',
    version: '1.0.0'
  },
  {
    capabilities: {
      tools: {}
    }
  }
);

// 工具 1: 掃描專案結構
server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [
      {
        name: 'scan_project',
        description: '掃描專案目錄結構',
        inputSchema: {
          type: 'object',
          properties: {
            rootPath: {
              type: 'string',
              description: '專案根目錄路徑'
            }
          },
          required: ['rootPath']
        }
      },
      {
        name: 'analyze_dependencies',
        description: '分析專案依賴關係',
        inputSchema: {
          type: 'object',
          properties: {
            rootPath: {
              type: 'string',
              description: '專案根目錄路徑'
            }
          },
          required: ['rootPath']
        }
      }
    ]
  };
});

// 處理工具呼叫
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  try {
    if (name === 'scan_project') {
      const structure = await getDirectoryStructure(args.rootPath);
      return {
        content: [{
          type: 'text',
          text: JSON.stringify(structure, null, 2)
        }]
      };
    }

    if (name === 'analyze_dependencies') {
      const deps = await analyzeDependencies(args.rootPath);
      return {
        content: [{
          type: 'text',
          text: JSON.stringify(deps, null, 2)
        }]
      };
    }

    throw new Error(`Unknown tool: ${name}`);
  } catch (error) {
    return {
      content: [{
        type: 'text',
        text: `Error: ${error.message}`
      }],
      isError: true
    };
  }
});

// 遞迴掃描目錄
async function getDirectoryStructure(dirPath: string, maxDepth = 3) {
  const entries = await fs.readdir(dirPath, { withFileTypes: true });
  const result = {};

  for (const entry of entries) {
    if (entry.isDirectory() && !entry.name.startsWith('.')) {
      const fullPath = path.join(dirPath, entry.name);
      result[entry.name] = await getDirectoryStructure(fullPath, maxDepth - 1);
    } else if (entry.isFile()) {
      result[entry.name] = 'file';
    }
  }

  return result;
}

// 分析依賴
async function analyzeDependencies(rootPath: string) {
  const packageJsonPath = path.join(rootPath, 'package.json');

  try {
    const content = await fs.readFile(packageJsonPath, 'utf-8');
    const pkg = JSON.parse(content);

    return {
      dependencies: Object.keys(pkg.dependencies || {}),
      devDependencies: Object.keys(pkg.devDependencies || {})
    };
  } catch {
    return { error: 'No package.json found' };
  }
}

// 啟動 Server
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error('Project Analyzer MCP Server running');
}

main().catch(console.error);
```

### 設定檔 (package.json)

```json
{
  "name": "project-analyzer-mcp-server",
  "version": "1.0.0",
  "type": "module",
  "main": "dist/index.js",
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.0.0"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "typescript": "^5.0.0"
  }
}
```

### TypeScript 設定 (tsconfig.json)

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*"]
}
```

### 在 Claude Desktop 中使用

```json
// ~/Library/Application Support/Claude/claude_desktop_config.json (macOS)
// %APPDATA%\Claude\claude_desktop_config.json (Windows)

{
  "mcpServers": {
    "project-analyzer": {
      "command": "node",
      "args": ["path/to/project-analyzer/dist/index.js"]
    }
  }
}
```

---

## 最佳實踐

### ✅ DO (應該做的事)

1. **明確的工具描述**
   ```typescript
   // ✅ Good
   {
     name: 'search_code',
     description: '在專案中搜尋特定程式碼片段，支援正則表達式'
   }

   // ❌ Bad
   {
     name: 'search',
     description: '搜尋'
   }
   ```

2. **完善的錯誤處理**
   ```typescript
   try {
     const result = await riskyOperation();
     return { content: [{ type: 'text', text: result }] };
   } catch (error) {
     return {
       content: [{
         type: 'text',
         text: `操作失敗: ${error.message}`
       }],
       isError: true
     };
   }
   ```

3. **輸入驗證**
   ```typescript
   function validateInput(args: any) {
     if (!args.path) {
       throw new Error('路徑參數為必填');
     }
     if (!args.path.startsWith('/safe/directory')) {
       throw new Error('不允許存取此目錄');
     }
   }
   ```

### ❌ DON'T (不該做的事)

1. **不要忽視安全性**
   ```typescript
   // ❌ 危險：直接執行任意指令
   const result = eval(userInput);

   // ✅ 安全：使用白名單
   const allowedCommands = ['git', 'npm', 'ls'];
   if (!allowedCommands.includes(cmd)) {
     throw new Error('不允許的指令');
   }
   ```

2. **不要忽略效能**
   ```typescript
   // ❌ 讀取整個大檔案到記憶體
   const content = fs.readFileSync('huge-file.log');

   // ✅ 使用串流處理
   const stream = fs.createReadStream('huge-file.log');
   ```

3. **不要回傳過多資料**
   ```typescript
   // ❌ 回傳整個專案樹
   return entireProjectTree;

   // ✅ 只回傳相關部分
   return {
     path: args.path,
    children: relevantChildren.slice(0, 100)
   };
   ```

---

## 進階技巧

### 1. 快取機制

```typescript
const cache = new Map();

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const cacheKey = JSON.stringify(request.params);

  if (cache.has(cacheKey)) {
    return cache.get(cacheKey);
  }

  const result = await expensiveOperation(request.params);
  cache.set(cacheKey, result);
  return result;
});
```

### 2. 非同步處理

```typescript
// 使用 SSE (Server-Sent Events) 回傳進度
async function* longRunningTask() {
  yield { status: 'processing', progress: 0 };

  // 執行工作...

  yield { status: 'completed', progress: 100 };
}
```

### 3. 工具鏈 (Tool Chaining)

```typescript
// 讓 AI 依序呼叫多個工具完成複雜任務
if (name === 'full_analysis') {
  const structure = await scanProject(args.rootPath);
  const deps = await analyzeDependencies(args.rootPath);
  const tests = await runTests(args.rootPath);

  return {
    content: [{
      type: 'text',
      text: JSON.stringify({
        structure,
        dependencies: deps,
        testResults: tests
      }, null, 2)
    }]
  };
}
```

---

## 🎯 實際應用場景

### 場景 1：自動化程式碼審查
```typescript
// MCP Server 提供：
// 1. 讀取 PR 檔案
// 2. 執行 linter
// 3. 執行測試
// 4. 產生審查報告
```

### 場景 2：智慧除錯助理
```typescript
// MCP Server 提供：
// 1. 讀取 log 檔案
// 2. 過濾錯誤訊息
// 3. 分析錯誤模式
// 4. 建議修復方案
```

### 場景 3：文件生成器
```typescript
// MCP Server 提供：
// 1. 掃描程式碼註解
// 2. 分析函式簽章
// 3. 產生 API 文件
// 4. 產生使用範例
```

---

## 參考資源

- [MCP 官方文件](https://modelcontextprotocol.io/)
- [MCP SDK GitHub](https://github.com/modelcontextprotocol/typescript-sdk)
- [Claude Desktop MCP 設定指南](https://github.com/anthropics/anthropic-quickstarts/tree/main/mcp-server)

---

## 下一步

閱讀下一篇：[**02-prompt-engineering.md**](./02-prompt-engineering.md)
