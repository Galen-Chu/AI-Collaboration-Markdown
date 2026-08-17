# Tool Calling 應用場景實戰

> 本文結合我的 G-Code 生態系實際案例，展示 Tool Calling 如何在
> 多層 Agent 架構、CI/CD 管線、和 MCP 整合中運作。

## 📚 目錄
- [什麼是 Tool Calling？](#什麼是-tool-calling)
- [為什麼需要 Tool Calling？](#為什麼需要-tool-calling)
- [核心應用場景](#核心應用場景)
- [實作案例](#實作案例)
- [進階技巧](#進階技巧)
- [最佳實踐](#最佳實踐)

---

## 什麼是 Tool Calling？

**Tool Calling** 是讓 AI 模型能夠「執行動作」而非僅「產生文字」的能力。透過工具呼叫，AI 可以：

- 🌐 **發送 HTTP 請求**：查詢 API、獲取即時資料
- 💾 **存取資料庫**：讀取、寫入、更新資料
- 📁 **操作檔案系統**：讀取檔案、掃描目錄
- 🔧 **執行系統指令**：執行測試、部署流程
- 📊 **處理資料**：分析、轉換、視覺化

### 傳統 vs Tool Calling

| 傳統對話 | Tool Calling |
|---------|--------------|
| AI：「建議使用 `axios` 發送請求」 | AI：**實際執行** HTTP 請求並傳回結果 |
| AI：「你需要執行 `npm test`」 | AI：**自動執行**測試並告訴你結果 |
| AI：「請手動檢查資料庫」 | AI：**直接查詢**資料庫並分析結果 |

---

## 為什麼需要 Tool Calling？

### 問題場景 1：即時資料查詢

❌ **傳統方式：**
```
用戶：現在比特幣價格是多少？
AI：我無法提供即時資訊，建議查看加密貨幣網站...
```

✅ **Tool Calling 方式：**
```
用戶：現在比特幣價格是多少？
AI：[呼叫 Crypto API] 當前比特幣價格為 $42,567.89，
過去 24 小時上漲 3.2%...
```

### 問題場景 2：自動化工作流程

❌ **傳統方式：**
```
用戶：幫我部署到生產環境
AI：以下是部署步驟：
1. 執行 npm run build
2. 上傳到伺服器
3. 重啟服務
（用戶需要手動執行）
```

✅ **Tool Calling 方式：**
```
用戶：幫我部署到生產環境
AI：[執行 build] → [上傳到伺服器] → [重啟服務]
✅ 部署完成！版本 v1.2.3 已成功上線。
```

### 問題場景 3：多步驟資料分析

❌ **傳統方式：**
```
用戶：分析這個月的銷售數據
AI：請提供銷售數據...
（用戶需要手動匯出 CSV、上傳、等待分析）
```

✅ **Tool Calling 方式：**
```
用戶：分析這個月的銷售數據
AI：[查詢資料庫] → [聚合數據] → [產生圖表] → [提供洞察]
本月銷售額 $125K，比上月成長 18%...
```

---

## 核心應用場景

### 場景 1：API 整合與查詢

讓 AI 能夠查詢外部 API 並提供即時資訊。

```typescript
// 範例：天氣查詢工具
{
  name: 'get_weather',
  description: '查詢指定城市的天氣預報',
  inputSchema: {
    type: 'object',
    properties: {
      city: {
        type: 'string',
        description: '城市名稱'
      },
      units: {
        type: 'string',
        enum: ['celsius', 'fahrenheit'],
        description: '溫度單位'
      }
    },
    required: ['city']
  },
  handler: async (args) => {
    // 呼叫天氣 API
    const response = await fetch(
      `https://api.weather.com/current?city=${args.city}&units=${args.units}`
    );
    const data = await response.json();

    return {
      temperature: data.temp,
      condition: data.condition,
      humidity: data.humidity,
      forecast: data.forecast
    };
  }
}
```

**實際應用：**
- 🌤️ 天氣查詢
- 💰 股價查詢
- 📦 物流追蹤
- 🔍 新聞搜尋

### 場景 2：資料庫操作

讓 AI 能夠直接查詢與操作資料庫。

```typescript
// 範例：安全查詢工具
{
  name: 'query_database',
  description: '執行安全的資料庫查詢',
  inputSchema: {
    type: 'object',
    properties: {
      queryType: {
        type: 'string',
        enum: ['select', 'aggregate', 'search'],
        description: '查詢類型'
      },
      table: {
        type: 'string',
        description: '資料表名稱'
      },
      conditions: {
        type: 'object',
        description: '查詢條件'
      }
    },
    required: ['queryType', 'table']
  },
  handler: async (args) => {
    // 使用白名單機制防止 SQL Injection
    const allowedTables = ['users', 'products', 'orders'];
    if (!allowedTables.includes(args.table)) {
      throw new Error(`不允許存取資料表: ${args.table}`);
    }

    // 使用 ORM 或參數化查詢
    switch (args.queryType) {
      case 'select':
        return await db.table(args.table)
          .where(args.conditions)
          .select();
      case 'aggregate':
        return await db.table(args.table)
          .where(args.conditions)
          .sum('amount as total');
      case 'search':
        return await db.table(args.table)
          .search(args.conditions.keyword);
    }
  }
}
```

**實際應用：**
- 📊 業務報表生成
- 🔍 資料查詢
- 📈 趨勢分析
- 👥 用戶管理

### 場景 3：檔案系統操作

讓 AI 能夠讀取與分析本地檔案。

```typescript
// 範例：檔案分析工具
{
  name: 'analyze_project',
  description: '分析專案結構與程式碼品質',
  inputSchema: {
    type: 'object',
    properties: {
      rootPath: {
        type: 'string',
        description: '專案根目錄'
      },
      analysisType: {
        type: 'string',
        enum: ['structure', 'dependencies', 'complexity', 'all'],
        description: '分析類型'
      }
    },
    required: ['rootPath']
  },
  handler: async (args) => {
    const results = {};

    if (args.analysisType === 'structure' || args.analysisType === 'all') {
      results.structure = await analyzeDirectoryStructure(args.rootPath);
    }

    if (args.analysisType === 'dependencies' || args.analysisType === 'all') {
      results.dependencies = await analyzeDependencies(args.rootPath);
    }

    if (args.analysisType === 'complexity' || args.analysisType === 'all') {
      results.complexity = await analyzeCodeComplexity(args.rootPath);
    }

    return results;
  }
}
```

**實際應用：**
- 📁 專案結構分析
- 🔍 程式碼審查
- 📦 依賴分析
- 📝 自動文件生成

### 場景 4：CI/CD 整合

讓 AI 參與自動化部署流程。

```typescript
// 範例：部署工具
{
  name: 'deploy_application',
  description: '部署應用到指定環境',
  inputSchema: {
    type: 'object',
    properties: {
      environment: {
        type: 'string',
        enum: ['staging', 'production'],
        description: '部署環境'
      },
      version: {
        type: 'string',
        description: '版本號碼'
      },
      skipTests: {
        type: 'boolean',
        description: '是否跳過測試'
      }
    },
    required: ['environment']
  },
  handler: async (args) => {
    const steps = [];

    // 步驟 1: 執行測試
    if (!args.skipTests) {
      steps.push({ name: 'Running tests', action: 'npm test' });
      const testResults = await runTests();
      if (!testResults.success) {
        return { success: false, error: 'Tests failed', steps };
      }
    }

    // 步驟 2: 建置
    steps.push({ name: 'Building', action: 'npm run build' });
    await buildProject();

    // 步驟 3: 部署
    steps.push({ name: 'Deploying', action: `deploy to ${args.environment}` });
    await deploy(args.environment, args.version);

    return {
      success: true,
      environment: args.environment,
      version: args.version,
      url: `https://${args.environment}.example.com`,
      steps
    };
  }
}
```

**實際應用：**
- 🚀 自動化部署
- 🧪 測試執行
- 🔄 版本發布
- 📊 部署監控

### 場景 5：監控與警報

讓 AI 能夠監控系統並主動通知。

```typescript
// 範例：監控工具
{
  name: 'check_system_health',
  description: '檢查系統健康狀態',
  inputSchema: {
    type: 'object',
    properties: {
      services: {
        type: 'array',
        items: { type: 'string' },
        description: '要檢查的服務列表'
      },
      alertThreshold: {
        type: 'number',
        description: '警報閾值（CPU/記憶體使用率）'
      }
    }
  },
  handler: async (args) => {
    const healthReport = {
      timestamp: new Date().toISOString(),
      services: [],
      alerts: []
    };

    for (const service of args.services) {
      const status = await checkServiceHealth(service);
      healthReport.services.push({
        name: service,
        status: status.healthy ? 'healthy' : 'unhealthy',
        metrics: status.metrics
      });

      // 檢查是否超過閾值
      if (status.metrics.cpu > args.alertThreshold) {
        healthReport.alerts.push({
          severity: 'warning',
          service: service,
          message: `CPU usage ${status.metrics.cpu}% exceeds threshold ${args.alertThreshold}%`
        });
      }
    }

    return healthReport;
  }
}
```

**實際應用：**
- 📈 效能監控
- 🔔 錯誤通知
- 💾 容量規劃
- 📊 健康檢查

---

## 實作案例

### 案例 1：智慧客戶服務機器人

整合多個工具提供完整的客戶服務體驗。

```typescript
// 定義工具集
const customerServiceTools = [
  // 工具 1: 查詢訂單狀態
  {
    name: 'get_order_status',
    description: '查詢訂單的當前狀態',
    inputSchema: {
      type: 'object',
      properties: {
        orderId: {
          type: 'string',
          description: '訂單編號'
        }
      },
      required: ['orderId']
    },
    handler: async (args) => {
      const order = await db.orders.findByPK(args.orderId);
      return {
        status: order.status,
        estimatedDelivery: order.estimatedDelivery,
        trackingNumber: order.trackingNumber,
        items: order.items
      };
    }
  },

  // 工具 2: 查詢產品資訊
  {
    name: 'get_product_info',
    description: '查詢產品的詳細資訊',
    inputSchema: {
      type: 'object',
      properties: {
        productId: {
          type: 'string',
          description: '產品 ID'
        }
      },
      required: ['productId']
    },
    handler: async (args) => {
      const product = await db.products.findByPK(args.productId);
      return {
        name: product.name,
        price: product.price,
        stock: product.stock,
        description: product.description,
        specifications: product.specifications
      };
    }
  },

  // 工具 3: 處理退貨申請
  {
    name: 'process_return',
    description: '處理退貨申請',
    inputSchema: {
      type: 'object',
      properties: {
        orderId: {
          type: 'string',
          description: '訂單編號'
        },
        reason: {
          type: 'string',
          description: '退貨原因'
        }
      },
      required: ['orderId', 'reason']
    },
    handler: async (args) => {
      // 驗證訂單是否符合退貨條件
      const order = await db.orders.findByPK(args.orderId);
      if (!order.canReturn) {
        return { success: false, reason: '此訂單不符合退貨條件' };
      }

      // 建立退貨申請
      const returnRequest = await db.returns.create({
        orderId: args.orderId,
        reason: args.reason,
        status: 'pending'
      });

      return {
        success: true,
        returnId: returnRequest.id,
        instructions: '請將商品寄回至...'
      };
    }
  },

  // 工具 4: 查詢優惠券
  {
    name: 'check_coupon',
    description: '檢查優惠券是否可用',
    inputSchema: {
      type: 'object',
      properties: {
        code: {
          type: 'string',
          description: '優惠券代碼'
        }
      },
      required: ['code']
    },
    handler: async (args) => {
      const coupon = await db.coupons.findByCode(args.code);
      if (!coupon) {
        return { valid: false, reason: '優惠券不存在' };
      }

      if (coupon.expired) {
        return { valid: false, reason: '優惠券已過期' };
      }

      return {
        valid: true,
        discount: coupon.discount,
        description: coupon.description
      };
    }
  }
];

// 使用範例
```

**對話範例：**
```
用戶：我的訂單 #12345 什麼時候會到？

AI：[呼叫 get_order_status]
您的訂單 #12345 目前狀態為「運送中」，
預計於 3 天後（10/8）送達。
物流追蹤號碼：TW987654321

用戶：我想申請退貨

AI：[呼叫 process_return]
我了解您想申請退貨。
請告訴我退貨原因，我會協助您處理。

用戶：商品有瑕疵

AI：[呼叫 process_return with reason="商品有瑕疵"]
您的退貨申請已建立（退貨單 #R67890）。
請將商品寄回至：
台北市信義區忠孝東路四段 123 號
我們會在收到商品後 3 個工作天內完成退款。
```

### 案例 2：自動化資料分析助手

整合資料查詢、分析、視覺化工具。

```typescript
// 定義資料分析工具集
const analyticsTools = [
  // 工具 1: 查詢銷售數據
  {
    name: 'query_sales_data',
    description: '查詢指定時間範圍的銷售數據',
    inputSchema: {
      type: 'object',
      properties: {
        startDate: { type: 'string', format: 'date' },
        endDate: { type: 'string', format: 'date' },
        groupBy: {
          type: 'string',
          enum: ['day', 'week', 'month', 'category', 'product']
        }
      },
      required: ['startDate', 'endDate']
    },
    handler: async (args) => {
      const data = await db.orders
        .whereBetween('createdAt', [args.startDate, args.endDate])
        .groupBy(args.groupBy)
        .select();

      return data;
    }
  },

  // 工具 2: 計算統計指標
  {
    name: 'calculate_metrics',
    description: '計算銷售統計指標',
    inputSchema: {
      type: 'object',
      properties: {
        data: {
          type: 'array',
          description: '銷售數據'
        },
        metrics: {
          type: 'array',
          items: {
            type: 'string',
            enum: ['total', 'average', 'median', 'growth_rate', 'conversion_rate']
          }
        }
      }
    },
    handler: async (args) => {
      const results = {};
      for (const metric of args.metrics) {
        switch (metric) {
          case 'total':
            results.total = args.data.reduce((sum, d) => sum + d.amount, 0);
            break;
          case 'average':
            results.average = args.data.reduce((sum, d) => sum + d.amount, 0) / args.data.length;
            break;
          // ... 更多指標
        }
      }
      return results;
    }
  },

  // 工具 3: 產生圖表
  {
    name: 'generate_chart',
    description: '產生資料視覺化圖表',
    inputSchema: {
      type: 'object',
      properties: {
        type: {
          type: 'string',
          enum: ['line', 'bar', 'pie', 'scatter']
        },
        data: { type: 'array' },
        title: { type: 'string' }
      }
    },
    handler: async (args) => {
      // 使用圖表函式庫產生圖表
      const chart = createChart(args.type, args.data, args.title);
      const imageUrl = await uploadChart(chart);
      return { imageUrl, chartData: args.data };
    }
  },

  // 工具 4: 產生報告
  {
    name: 'generate_report',
    description: '產生分析報告',
    inputSchema: {
      type: 'object',
      properties: {
        title: { type: 'string' },
        sections: {
          type: 'array',
          items: { type: 'string' },
          description: '報告章節'
        },
        includeCharts: { type: 'boolean' }
      }
    },
    handler: async (args) => {
      const report = {
        title: args.title,
        generatedAt: new Date().toISOString(),
        sections: []
      };

      for (const section of args.sections) {
        // 生成各個章節內容
      }

      return report;
    }
  }
];
```

**使用範例：**
```
用戶：分析這個月的銷售表現

AI：[呼叫 query_sales_data]
[呼叫 calculate_metrics]
[呼叫 generate_chart]

## 10 月份銷售分析報告

### 整體表現
- 總銷售額：$125,000
- 平均日銷售：$4,032
- 成長率：+18.2% vs 上月

### 熱銷產品 Top 5
1. 產品 A - $25,000 (20%)
2. 產品 B - $18,750 (15%)
3. 產品 C - $15,000 (12%)
...

### 趨勢分析
[附上趨勢圖]

### 洞察與建議
- 週末銷售量較平日高 35%，建議加強週末行銷
- 產品 A 的庫存不足，建議補貨
- 新用戶轉化率提升至 12%，推廣效果顯著
```

### 案例 3：DevOps 自動化助理

整合多個 DevOps 工具實現自動化運維。

```typescript
const devOpsTools = [
  // 工具 1: 檢查伺服器狀態
  {
    name: 'check_servers',
    description: '檢查所有伺服器的健康狀態',
    inputSchema: {
      type: 'object',
      properties: {
        environment: {
          type: 'string',
          enum: ['staging', 'production', 'all']
        }
      }
    },
    handler: async (args) => {
      const servers = await getServers(args.environment);
      const results = [];

      for (const server of servers) {
        const health = await checkServerHealth(server);
        results.push({
          name: server.name,
          status: health.healthy ? 'healthy' : 'unhealthy',
          cpu: health.cpu,
          memory: health.memory,
          disk: health.disk
        });
      }

      return results;
    }
  },

  // 工具 2: 執行部署
  {
    name: 'deploy',
    description: '部署應用到指定環境',
    inputSchema: {
      type: 'object',
      properties: {
        environment: { type: 'string' },
        version: { type: 'string' },
        strategy: {
          type: 'string',
          enum: ['rolling', 'blue-green', 'canary']
        }
      }
    },
    handler: async (args) => {
      const deployment = await createDeployment(args);
      const result = await executeDeployment(deployment);
      return result;
    }
  },

  // 工具 3: 查看日誌
  {
    name: 'view_logs',
    description: '查看應用程式日誌',
    inputSchema: {
      type: 'object',
      properties: {
        service: { type: 'string' },
        startTime: { type: 'string' },
        endTime: { type: 'string' },
        level: {
          type: 'string',
          enum: ['error', 'warn', 'info', 'debug']
        }
      }
    },
    handler: async (args) => {
      const logs = await queryLogs({
        service: args.service,
        from: args.startTime,
        to: args.endTime,
        level: args.level
      });

      return {
        total: logs.length,
        logs: logs.slice(0, 100) // 限制回傳數量
      };
    }
  },

  // 工具 4: 擴展服務
  {
    name: 'scale_service',
    description: '擴展或縮減服務實例數量',
    inputSchema: {
      type: 'object',
      properties: {
        service: { type: 'string' },
        replicas: { type: 'number', minimum: 1 }
      }
    },
    handler: async (args) => {
      const result = await scaleService(args.service, args.replicas);
      return {
        service: args.service,
        previousReplicas: result.previous,
        currentReplicas: result.current,
        status: 'scaling'
      };
    }
  }
];
```

**使用範例：**
```
用戶：生產環境出現問題了，幫我檢查一下

AI：[呼叫 check_servers]
[呼叫 view_logs]

我發現生產環境的 API 伺服器異常：

### 伺服器狀態
- api-server-1: ❌ CPU 95%, 記憶體 87%
- api-server-2: ⚠️ CPU 78%, 記憶體 65%
- api-server-3: ✅ 正常

### 錯誤日誌（最近 10 分鐘）
ERROR [10:23:45] Database connection timeout
ERROR [10:24:12] Request timeout after 30s
ERROR [10:25:01] Too many connections

### 建議措施
1. 立即擴展 API 伺服器至 5 個實例
2. 檢查資料庫連線池配置
3. 重啟 api-server-1

要執行這些措施嗎？

用戶：執行

AI：[呼叫 scale_service]
[呼叫 restart_server]

✅ 已擴展至 5 個實例
✅ 已重啟 api-server-1

系統已恢復正常，當前請求延遲：120ms
```

---

## 進階技巧

### 技巧 1：工具鏈 (Tool Chaining)

串接多個工具完成複雜任務。

```typescript
// 範例：智慧報告生成流程
async function generateSmartReport() {
  // 步驟 1: 收集資料
  const salesData = await tools.query_sales_data({
    startDate: '2025-01-01',
    endDate: '2025-01-31'
  });

  // 步驟 2: 分析數據
  const metrics = await tools.calculate_metrics({
    data: salesData,
    metrics: ['total', 'average', 'growth_rate']
  });

  // 步驟 3: 產生圖表
  const charts = await tools.generate_chart({
    type: 'line',
    data: salesData,
    title: '1 月份銷售趨勢'
  });

  // 步驟 4: 產生報告
  const report = await tools.generate_report({
    title: '1 月份銷售報告',
    sections: ['總結', '詳細分析', '建議'],
    includeCharts: true
  });

  return report;
}
```

### 技巧 2：條件式工具呼叫

根據情境選擇適當的工具。

```typescript
async function intelligentQuery(userQuery) {
  // 分析用戶意圖
  const intent = await analyzeIntent(userQuery);

  // 根據意圖選擇工具
  switch (intent.type) {
    case 'order_inquiry':
      return await tools.get_order_status({ orderId: intent.orderId });
    case 'product_inquiry':
      return await tools.get_product_info({ productId: intent.productId });
    case 'complaint':
      return await tools.create_ticket({ issue: intent.description });
    default:
      return await tools.general_search({ query: userQuery });
  }
}
```

### 技巧 3：並行工具執行

同時執行多個獨立工具提升效率。

```typescript
async function parallelHealthCheck() {
  const results = await Promise.all([
    tools.check_servers({ environment: 'production' }),
    tools.check_database(),
    tools.check_external_apis()
  ]);

  return {
    servers: results[0],
    database: results[1],
    apis: results[2]
  };
}
```

### 技巧 4：錯誤處理與重試

實作強健的錯誤處理機制。

```typescript
async function executeToolWithRetry(toolName, args, maxRetries = 3) {
  let lastError;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const result = await tools[toolName](args);
      return { success: true, result };
    } catch (error) {
      lastError = error;
      console.error(`Attempt ${attempt} failed:`, error.message);

      if (attempt < maxRetries) {
        // 指數退避
        await sleep(Math.pow(2, attempt) * 1000);
      }
    }
  }

  return {
    success: false,
    error: lastError.message,
    attempts: maxRetries
  };
}
```

---

## 最佳實踐

### ✅ 應該做的事

1. **明確的工具定義**
   ```typescript
   // ✅ Good
   {
     name: 'search_users',
     description: '搜尋用戶資料，支援按姓名、Email、ID 查詢',
     inputSchema: {
       type: 'object',
       properties: {
         query: {
           type: 'string',
           description: '搜尋關鍵字（姓名、Email 或用戶 ID）'
         }
       }
     }
   }
   ```

2. **完善的輸入驗證**
   ```typescript
   // ✅ Good
   handler: async (args) => {
     // 驗證輸入
     if (!args.email || !isValidEmail(args.email)) {
       throw new Error('無效的 Email 格式');
     }

     // 限制權限
     if (args.table && !isAuthorizedTable(args.table)) {
       throw new Error('無權存取此資料表');
     }

     // 執行操作
     return await operation(args);
   }
   ```

3. **清晰的錯誤訊息**
   ```typescript
   // ✅ Good
   return {
     success: false,
     error: {
       code: 'INSUFFICIENT_PERMISSIONS',
       message: '您沒有權限執行此操作',
       requiredPermission: 'admin.write',
       currentPermission: 'user.read'
     }
   };
   ```

### ❌ 不該做的事

1. **不要忽視安全性**
   ```typescript
   // ❌ Bad: 直接執行用戶輸入
   handler: async (args) => {
     const result = await db.raw(args.sql); // SQL Injection 風險！
     return result;
   }

   // ✅ Good: 使用參數化查詢
   handler: async (args) => {
     const result = await db
       .table('users')
       .where('email', args.email)
       .first();
     return result;
   }
   ```

2. **不要暴露敏感資訊**
   ```typescript
   // ❌ Bad
   return {
     password: user.password, // 不要回傳密碼！
     apiKey: config.apiKey
   };

   // ✅ Good
   return {
     id: user.id,
     name: user.name,
     email: user.email
     // 敏感欄位自動過濾
   };
   ```

3. **不要忽略資源限制**
   ```typescript
   // ❌ Bad: 可能回傳大量資料
   handler: async (args) => {
     return await db.table('logs').selectAll(); // 可能有數百萬筆！
   }

   // ✅ Good: 加上限制
   handler: async (args) => {
     return await db.table('logs')
       .limit(100)
       .orderBy('createdAt', 'desc')
       .select();
   }
   ```

---

## 安全性考量

### 1. 權限控制

```typescript
// 實作角色基礎的權限控制
const permissions = {
  admin: ['read', 'write', 'delete'],
  user: ['read'],
  guest: ['read:public']
};

function checkPermission(user, requiredPermission) {
  const userPermissions = permissions[user.role];
  return userPermissions.includes(requiredPermission);
}
```

### 2. 輸入消毒

```typescript
// 消毒用戶輸入
function sanitizeInput(input) {
  return {
    ...input,
    // 移除潛在危險字元
    query: input.query?.replace(/[;'"]/g, ''),
    // 限制長度
    text: input.text?.substring(0, 1000)
  };
}
```

### 3. 速率限制

```typescript
// 防止工具被濫用
const rateLimiter = new RateLimiter({
  windowMs: 60000, // 1 分鐘
  maxRequests: 100 // 最多 100 次請求
});

async function callToolWithRateLimit(toolName, args) {
  await rateLimiter.check(toolName);
  return await tools[toolName](args);
}
```

---

## 效能優化

### 1. 快取機制

```typescript
const cache = new LRUCache({
  max: 100,
  ttl: 60000 // 1 分鐘過期
});

async function cachedToolCall(toolName, args) {
  const cacheKey = `${toolName}:${JSON.stringify(args)}`;

  // 檢查快取
  if (cache.has(cacheKey)) {
    return cache.get(cacheKey);
  }

  // 執行工具
  const result = await tools[toolName](args);

  // 存入快取
  cache.set(cacheKey, result);

  return result;
}
```

### 2. 批次處理

```typescript
// 批次處理多個請求
async function batchQuery(items) {
  const results = await Promise.all(
    items.map(item => tools.query(item))
  );

  return results;
}
```

### 3. 懶惰載入

```typescript
// 只在需要時才載入資源
async function lazyLoadTool(toolName) {
  if (!loadedTools[toolName]) {
    loadedTools[toolName] = await import(`./tools/${toolName}.js`);
  }

  return loadedTools[toolName];
}
```

---

## 測試策略

### 單元測試

```typescript
describe('Tool: query_database', () => {
  it('should return data for valid query', async () => {
    const result = await tools.query_database({
      table: 'users',
      conditions: { status: 'active' }
    });

    expect(result).toBeDefined();
    expect(result.data).toBeInstanceOf(Array);
  });

  it('should throw error for unauthorized table', async () => {
    await expect(
      tools.query_database({ table: 'internal_data' })
    ).rejects.toThrow('Unauthorized');
  });
});
```

### 整合測試

```typescript
describe('Tool Chain: Order Processing', () => {
  it('should process order end-to-end', async () => {
    // 1. 建立訂單
    const order = await tools.create_order({
      items: [{ productId: 'p1', quantity: 2 }]
    });

    // 2. 處理付款
    const payment = await tools.process_payment({
      orderId: order.id,
      amount: order.total
    });

    // 3. 更新庫存
    await tools.update_inventory({
      items: order.items
    });

    // 驗證結果
    const updatedOrder = await tools.get_order({ id: order.id });
    expect(updatedOrder.status).toBe('paid');
  });
});
```

---

## 參考資源

- [MCP Tool Calling 規範](https://modelcontextprotocol.io/docs/concepts/tools/)
- [OpenAI Function Calling](https://platform.openai.com/docs/guides/function-calling)
- [Anthropic Tool Use](https://docs.anthropic.com/claude/docs/tool-use)

---

## 我的生態系實戰案例

### 案例 1：G-Code 專案 CI/CD 整合（program-g-code）

在 [program-g-code](https://github.com/Galen-Chu/program-g-code) 中，Tool Calling 體現在閉環自測試：

```
GitHub Actions 觸發
  → 呼叫工具包的 lint.sh（auto-detect Node.js/Python）
  → 呼叫 test.sh（偵測 Jest/pytest 並執行）
  → 呼叫 build.sh --dry-run（模擬建置）
  → 呼叫 deploy.sh --dry-run（模擬部署）
  → 呼叫 health-check.sh（驗證 mock server）
```

工具包本身不判斷專案類型——AI 或使用者透過 `config/ci-cd.conf` 的
`target_projects` 參數指定目標，工具包的腳本 auto-detect 並執行。

### 案例 2：Agent 層級間的 Tool 委派（AI-Agent-Skill）

在 [AI-Agent-Skill](https://github.com/Galen-Chu/AI-Agent-Skill) 的四層架構中，
Tool Calling 就是層級之間的委派機制：

```
orch-main（① 協調層）
  → Agent tool → wf-news-digest（② 工作流層）
    → Agent tool → mod-web-scraper（③ 能力模組層）
      → WebFetch/WebSearch → 外部資料來源
```

**關鍵設計**：③ 層的能力模組不判斷「誰在呼叫我」——它只知道自己該做什麼。
這使得同一個 `mod-summarizer` 可以被新聞摘要、學術審查、財報分析等多個
工作流呼叫，不需要任何修改。

### 案例 3：Pipeline 評估閘門（AI-Pipeline-Hook + AI-Eval-Rubric）

在排程系統中，評估是一種「Tool Calling 的 Tool Calling」：

```yaml
# pipelines/daily-news-report.yaml 的一個步驟
steps:
  - id: evaluate_output
    type: claude-skill
    skill: mod-eval-report          # AI-Agent-Skill 中的評估模組
    rubric: documentation            # AI-Eval-Rubric 中的評估規格
```

```
Pipeline Runner（AI-Pipeline-Hook）
  → 呼叫 mod-eval-report（AI-Agent-Skill）
    → 讀取 documentation rubric（AI-Eval-Rubric）
      → 比對產出與規格
        → 產出 pass/fail 報告
```

### 案例 4：Obsidian MCP 的讀寫整合

在 [Obsidian Journal](https://github.com/Galen-Chu/Obsidian_Journal) 中，
`personal-assistant` Agent 透過 Obsidian MCP 寫入日誌：

```
personal-assistant（① 協調層）
  → Obsidian MCP（Local REST API）
    → 寫入 daily/life/2026-08-17.md
    → 遵循 obsidian-note-conventions Skill 的格式
```

**已知限制**：Obsidian MCP 需要 Obsidian 應用程式保持執行中（127.0.0.1 REST API），
因此只適合本機觸發的 pipeline，不適合雲端排程。

---

## 總結

Tool Calling 讓 AI 從「被動回應者」變成「主動執行者」：

- 🎯 **精準性**：直接呼叫工具取得準確結果
- 🚀 **效率**：自動化重複性任務
- 🔧 **整合性**：連接各種系統與服務
- 💪 **擴展性**：無限的工具整合可能

**關鍵成功因素：**
1. 清晰的工具定義與描述
2. 嚴格的輸入驗證與權限控制
3. 完善的錯誤處理機制
4. 持續的效能優化與測試
5. **跨專案的生態系整合**——工具應該可以在不同層級間重用

---

回到主目錄：[**README.md**](../README.md)
