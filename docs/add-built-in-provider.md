# 添加内置 AI 提供商指南

本文档记录如何为思源笔记 AI Copilot 插件添加新的内置 AI 提供商。

## 概述

添加一个新的内置 AI 提供商需要修改以下文件：

1. `src/ai-chat.ts` - 核心 API 配置
2. `src/defaultSettings.ts` - 默认设置
3. `src/SettingsPannel.svelte` - 设置面板
4. `src/components/ProviderConfigPanel.svelte` - 提供商配置面板
5. `src/components/MultiModelSelector.svelte` - 模型选择器
6. `src/index.ts` - 插件入口（设置迁移）
7. `i18n/zh_CN.json` - 中文翻译
8. `i18n/en_US.json` - 英文翻译

---

## 详细步骤

### 1. 添加 AIProvider 类型

**文件**: `src/ai-chat.ts`

在 `AIProvider` 类型中添加新的提供商标识符：

```typescript
export type AIProvider = 'gemini' | 'deepseek' | 'openai' | 'moonshot' | 'volcano' | 'Achuan' | 'zhipu' | 'custom';
```

### 2. 配置提供商 API 信息

**文件**: `src/ai-chat.ts`

在 `PROVIDER_CONFIGS` 中添加提供商配置：

```typescript
const PROVIDER_CONFIGS: Record<AIProvider, ProviderConfig> = {
    // ... 其他提供商
    zhipu: {
        name: '智谱AI',
        baseUrl: 'https://open.bigmodel.cn/api/paas/v4',
        modelsEndpoint: '/models',
        chatEndpoint: '/chat/completions',
        apiKeyHeader: 'Authorization',
        websiteUrl: 'https://open.bigmodel.cn/'
    },
    // ...
};
```

**配置项说明**:
- `name`: 提供商显示名称
- `baseUrl`: API 基础地址
- `modelsEndpoint`: 获取模型列表的端点
- `chatEndpoint`: 聊天 completions 端点
- `apiKeyHeader`: 认证头名称（通常为 `Authorization`）
- `websiteUrl`: 官网链接（用于引导用户获取 API Key）

### 3. 更新内置提供商列表

**文件**: `src/ai-chat.ts`

更新所有 `isBuiltIn` 判断的数组，添加新提供商：

```typescript
const isBuiltIn = ['gemini', 'deepseek', 'openai', 'moonshot', 'volcano', 'Achuan', 'zhipu'].includes(provider);
```

**需要修改的位置**（共 3 处）:
- `fetchModels` 函数
- `chat` 函数
- `generateImage` 函数

### 4. 添加默认设置

**文件**: `src/defaultSettings.ts`

在 `aiProviders` 中添加默认配置：

```typescript
aiProviders: {
    // ... 其他提供商
    zhipu: {
        apiKey: '',
        customApiUrl: '',
        models: []
    },
    customProviders: [] as CustomProviderConfig[]
}
```

### 5. 添加设置面板支持

**文件**: `src/SettingsPannel.svelte`

#### 5.1 添加内置平台名称

```typescript
const builtInProviderNames: Record<string, string> = {
    Achuan: t('platform.builtIn.Achuan'),
    gemini: t('platform.builtIn.gemini'),
    openai: t('platform.builtIn.openai'),
    deepseek: t('platform.builtIn.deepseek'),
    moonshot: t('platform.builtIn.moonshot'),
    volcano: t('platform.builtIn.volcano'),
    zhipu: t('platform.builtIn.zhipu'),  // 添加这一行
};
```

#### 5.2 添加默认 API 地址

```typescript
const builtInProviderDefaultUrls: Record<string, string> = {
    Achuan: 'https://gpt.achuan-2.top/',
    gemini: 'https://generativelanguage.googleapis.com',
    deepseek: 'https://api.deepseek.com',
    moonshot: 'https://api.moonshot.cn',
    openai: 'https://api.openai.com',
    volcano: 'https://ark.cn-beijing.volces.com',
    zhipu: 'https://open.bigmodel.cn/api/paas/v4',  // 添加这一行
};
```

#### 5.3 添加官网链接

```typescript
const builtInProviderWebsites: Record<string, string> = {
    Achuan: 'https://gpt.achuan-2.top/register?aff=ZndO',
    gemini: 'https://aistudio.google.com/apikey',
    deepseek: 'https://platform.deepseek.com/',
    moonshot: 'https://platform.moonshot.cn/',
    openai: 'https://platform.openai.com/',
    volcano: 'https://console.volcengine.com/ark',
    zhipu: 'https://open.bigmodel.cn/',  // 添加这一行
};
```

#### 5.4 添加初始化配置

在 `runload` 函数的 `aiProviders` 初始化中：

```typescript
if (!settings.aiProviders) {
    settings.aiProviders = {
        gemini: { apiKey: '', customApiUrl: '', models: [] },
        deepseek: { apiKey: '', customApiUrl: '', models: [] },
        openai: { apiKey: '', customApiUrl: '', models: [] },
        moonshot: { apiKey: '', customApiUrl: '', models: [] },
        volcano: { apiKey: '', customApiUrl: '', models: [] },
        Achuan: { apiKey: '', customApiUrl: '', models: [] },
        zhipu: { apiKey: '', customApiUrl: '', models: [] },  // 添加这一行
        customProviders: [],
    };
}
```

#### 5.5 添加到内置平台 ID 列表

```typescript
const builtInPlatformIds = [
    'Achuan',
    'gemini',
    'deepseek',
    'openai',
    'moonshot',
    'volcano',
    'zhipu',  // 添加这一行
];
```

### 6. 更新提供商配置面板

**文件**: `src/components/ProviderConfigPanel.svelte`

在内置平台列表中添加新提供商：

```typescript
const builtInProviders = ['Achuan', 'gemini', 'deepseek', 'openai', 'moonshot', 'volcano', 'zhipu'];
```

### 7. 更新模型选择器

**文件**: `src/components/MultiModelSelector.svelte`

在 `builtInProviderNames` 中添加新提供商：

```typescript
const builtInProviderNames: Record<string, string> = {
    Achuan: t('platform.builtIn.Achuan'),
    gemini: t('platform.builtIn.gemini'),
    deepseek: t('platform.builtIn.deepseek'),
    openai: t('platform.builtIn.openai'),
    volcano: t('platform.builtIn.volcano'),
    moonshot: t('platform.builtIn.moonshot'),
    zhipu: t('platform.builtIn.zhipu'),  // 添加这一行
};
```

### 8. 更新插件入口（设置迁移）

**文件**: `src/index.ts`

在设置迁移逻辑中：

```typescript
const builtInProviders = ['Achuan', 'gemini', 'deepseek', 'openai', 'moonshot', 'volcano', 'zhipu'];
```

### 9. 添加国际化翻译

**文件**: `i18n/zh_CN.json`

```json
{
    "platform": {
        "builtIn": {
            "zhipu": "智谱AI"
        }
    }
}
```

**文件**: `i18n/en_US.json`

```json
{
    "platform": {
        "builtIn": {
            "zhipu": "Zhipu AI"
        }
    }
}
```

---

## API 格式说明

### OpenAI 兼容格式

大多数国内 AI 提供商（如智谱AI、DeepSeek、Moonshot）使用 OpenAI 兼容的 API 格式：

```bash
curl -X POST "https://open.bigmodel.cn/api/paas/v4/chat/completions" \
-H "Content-Type: application/json" \
-H "Authorization: Bearer YOUR_API_KEY" \
-d '{
    "model": "glm-5",
    "messages": [
        {"role": "system", "content": "你是一个有用的AI助手。"},
        {"role": "user", "content": "你好"}
    ],
    "temperature": 1.0,
    "stream": true
}'
```

**关键特征**:
- 端点: `/v1/chat/completions` 或 `/chat/completions`
- 认证: `Authorization: Bearer {api_key}`
- 支持流式输出 (`stream: true`)

### Gemini 原生格式

Google Gemini 使用原生格式，详见代码中的 `chatGeminiFormat` 函数。

### Claude 原生格式

Anthropic Claude 使用原生格式，详见代码中的 `chatClaudeFormat` 函数。

---

## 验证清单

添加完成后，请检查以下功能是否正常工作：

- [ ] 设置面板中显示新提供商
- [ ] 可以添加/配置 API Key
- [ ] 可以获取模型列表
- [ ] 模型选择器中显示该提供商的模型
- [ ] 可以进行对话
- [ ] 流式输出正常
- [ ] 多模型选择器中能显示该提供商

---

## 常见问题

### Q: 添加后设置面板没有显示新提供商？

A: 可能是以下原因：
1. i18n 翻译文件未正确加载 - 尝试重载插件或重启思源笔记
2. `builtInProviderNames` 中缺少新提供商
3. 设置初始化代码未添加新提供商

### Q: 模型选择器中看不到该提供商的模型？

A: 检查 `MultiModelSelector.svelte` 中的 `builtInProviderNames` 是否包含新提供商。

### Q: 可以获取模型列表但无法对话？

A: 检查 `ai-chat.ts` 中的 `isBuiltIn` 列表是否包含新提供商（共 3 处）。

---

## 示例：添加智谱AI

参考上述步骤，智谱AI 的配置如下：

- **providerId**: `zhipu`
- **name**: `智谱AI` / `Zhipu AI`
- **baseUrl**: `https://open.bigmodel.cn/api/paas/v4`
- **modelsEndpoint**: `/models`
- **chatEndpoint**: `/chat/completions`
- **websiteUrl**: `https://open.bigmodel.cn/`

智谱AI 使用标准的 OpenAI 兼容 API 格式，支持流式输出。
