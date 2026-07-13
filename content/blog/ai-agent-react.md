---
date: '2026-07-13T10:55:28+08:00'
draft: false
title: 'Ai Agent React'
---

## React 是什么？

```js
import 'dotenv/config';
import { ChatOpenAI } from '@langchain/openai';
import { tool } from '@langchain/core/tools';
import { HumanMessage, SystemMessage, ToolMessage } from '@langchain/core/messages';
import fs from 'node:fs/promises';
import { z } from 'zod';

const model = new ChatOpenAI({ 
  modelName: process.env.MODEL_NAME || "qwen-plus",
  apiKey: process.env.OPENAI_API_KEY,
  temperature: 0,
  configuration: {
      baseURL: process.env.OPENAI_BASE_URL,
  },
});

// tool()是 LangChain 提供的一个高阶函数（Factory Function），
// 用来把一个普通的 JavaScript 函数，包装成 LLM 可以理解并调用的“工具”。
// 第一个参数：真正执行的业务逻辑
// 第二个参数：工具的元数据（名字、描述、参数结构）
// 标准化，统一工具调用格式
// 自描述，自动生成 JSON Schema
// 可绑定，可被 bindTools()挂载到模型
const readFileTool = tool(
  async ({ filePath }) => {
    const content = await fs.readFile(filePath, 'utf-8');
    console.log(`  [工具调用] read_file("${filePath}") - 成功读取 ${content.length} 字节`);
    return `文件内容:\n${content}`;
  },
  {
    name: 'read_file',
    description: '用此工具来读取文件内容。当用户要求读取文件、查看代码、分析文件内容时，调用此工具。输入文件路径（可以是相对路径或绝对路径）。',
    schema: z.object({
      filePath: z.string().describe('要读取的文件路径'),
    }),
  }
);

const tools = [
  readFileTool
];

const modelWithTools = model.bindTools(tools);

const messages = [
  new SystemMessage(`你是一个代码助手，可以使用工具读取文件并解释代码。

工作流程：
1. 用户要求读取文件时，立即调用 read_file 工具
2. 等待工具返回文件内容
3. 基于文件内容进行分析和解释

可用工具：
- read_file: 读取文件内容（使用此工具来获取文件内容）
`),
  new HumanMessage('请读取 ./src/tool-file-read.mjs 文件内容并解释代码')
];

let response = await modelWithTools.invoke(messages);
// console.log(response);

messages.push(response);

// 只要模型返回了 tool_calls，就继续循环调用工具
while (response.tool_calls && response.tool_calls.length > 0) {
  
  console.log(`\n[检测到 ${response.tool_calls.length} 个工具调用]`);
  
  // 执行所有工具调用
  const toolResults = await Promise.all(
    response.tool_calls.map(async (toolCall) => {
      const tool = tools.find(t => t.name === toolCall.name);
      if (!tool) {
        return `错误: 找不到工具 ${toolCall.name}`;
      }
      
      console.log(`  [执行工具] ${toolCall.name}(${JSON.stringify(toolCall.args)})`);
      try {
        const result = await tool.invoke(toolCall.args);
        return result;
      } catch (error) {
        return `错误: ${error.message}`;
      }
    })
  );
  
  // 将工具结果添加到消息历史
  response.tool_calls.forEach((toolCall, index) => {
    messages.push(
      new ToolMessage({
        content: toolResults[index],
        tool_call_id: toolCall.id,
      })
    );
  });
  
  // 再次调用模型，传入工具结果
  response = await modelWithTools.invoke(messages);
}

console.log('\n[最终回复]');
console.log(response.content);

```

## 执行 node src/tool-file-read.mjs

[检测到 1 个工具调用]
  [执行工具] read_file({"filePath":"./src/tool-file-read.mjs"})
  [工具调用] read_file("./src/tool-file-read.mjs") - 成功读取 2542 字节



[最终回复]
这段代码 `./src/tool-file-read.mjs` 是一个基于 **LangChain（v0.3+）** 的 Node.js 脚本，用于构建一个具备「文件读取能力」的 AI 助手（LLM Agent），其核心目标是：  
✅ **自动识别用户请求中的文件读取意图 → 调用 `read_file` 工具读取指定路径文件 → 将内容返回给模型 → 生成自然语言解释。**

下面分模块详细解释：

---

### 🔧 1. 环境与依赖
```js
import 'dotenv/config'; // 加载 .env 文件（如 OPENAI_API_KEY, OPENAI_BASE_URL, MODEL_NAME）
import { ChatOpenAI } from '@langchain/openai'; // LangChain 官方 OpenAI 兼容模型封装（支持自定义 base URL，适配 Qwen、Ollama 等）
import { tool } from '@langchain/core/tools'; // 创建可被 LLM 调用的工具（Tool）的工厂函数
import { HumanMessage, SystemMessage, ToolMessage } from '@langchain/core/messages'; // 消息类型（系统/用户/工具响应）
import fs from 'node:fs/promises'; // Node.js 原生 Promise 版 fs（安全异步读文件）
import { z } from 'zod'; // 用于定义工具参数 Schema（强类型校验 + 自动生成 JSON Schema）
```

> ✅ 注意：此脚本使用的是 **ESM 模块语法**（`.mjs` 后缀），需在 `package.json` 中设置 `"type": "module"`。

---

### 🤖 2. 大模型初始化（支持本地/私有模型）
```js
const model = new ChatOpenAI({ 
  modelName: process.env.MODEL_NAME || "qwen-plus",
  apiKey: process.env.OPENAI_API_KEY,
  temperature: 0,
  configuration: {
      baseURL: process.env.OPENAI_BASE_URL, // 关键！支持非 OpenAI 的兼容 API（如 DashScope、Qwen API、Ollama）
  },
});
```
- 不依赖 OpenAI 官方服务，通过 `baseURL` 可对接：
  - 阿里云 DashScope（Qwen 系列）
  - Ollama（`http://localhost:11434/v1`）
  - 自建 vLLM / FastChat 等
- `temperature: 0` → 确保输出确定性（适合工具调用场景）

---

### 🛠️ 3. 定义 `read_file` 工具（核心逻辑）
```js
const readFileTool = tool(
  async ({ filePath }) => {
    const content = await fs.readFile(filePath, 'utf-8');
    console.log(`  [工具调用] read_file("${filePath}") - 成功读取 ${content.length} 字节`);
    return `文件内容:\n${content}`;
  },
  {
    name: 'read_file',
    description: '用此工具来读取文件内容。当用户要求读取文件、查看代码、分析文件内容时，调用此工具。输入文件路径（可以是相对路径或绝对路径）。',
    // schema 定义了工具的参数结构
    schema: z.object({
      filePath: z.string().describe('要读取的文件路径'),
    }),
  }
);
```

- ✅ **`tool()` 是 LangChain 的标准工具包装器**：
  - 第一个参数：实际执行的异步函数（接收 `filePath`，返回字符串内容）
  - 第二个参数：元数据（名称、描述、Zod Schema）
- 📌 Schema 作用：
  - LLM 调用前会做参数校验（如 `filePath` 必须是 string）
  - 自动生成符合 OpenAI Tool Calling 格式的 JSON Schema（供 LLM 理解如何调用）
- ⚠️ 安全提示：生产环境应增加路径白名单/沙箱限制（当前可读任意文件，存在风险）

---

### 🧩 4. 绑定工具到模型 & 构建消息流
```js
const tools = [readFileTool];
const modelWithTools = model.bindTools(tools); // 模型“知道”有哪些工具可用

const messages = [
  new SystemMessage(`你是一个代码助手...`), // 明确 Agent 角色 + 工作流程（3 步法）
  new HumanMessage('请读取 ./src/tool-file-read.mjs 文件内容并解释代码') // 初始用户请求
];
```
- System Message 是关键！它教会 LLM：
  - 何时调用 `read_file`（用户要求读文件时）
  - 必须立即调用（不拖延、不假设）
  - 如何处理工具返回结果（基于内容解释）

---

### 🔄 5. 工具调用循环（Agent 执行引擎）

```js
let response = await modelWithTools.invoke(messages);

while (response.tool_calls && response.tool_calls.length > 0) {
  console.log(`\n[检测到 ${response.tool_calls.length} 个工具调用]`);
  
  // 并行执行所有 tool_calls（支持多工具并发）
  const toolResults = await Promise.all(
    response.tool_calls.map(async (toolCall) => { /* ... */ })
  );
  
  // 将每个工具结果转为 ToolMessage，加入对话历史
  response.tool_calls.forEach((toolCall, index) => {
    messages.push(new ToolMessage({ content: toolResults[index], tool_call_id: toolCall.id }));
  });
  
  // 再次调用模型，让其基于工具结果生成最终回复
  response = await modelWithTools.invoke(messages);
}

console.log('\n[最终回复]');
console.log(response.content);
```

- 这是典型的 **ReAct（Reasoning + Acting）Agent 循环**：
  1. LLM 输出 `tool_calls`（结构化调用指令）
  2. 代码解析并由 agent `执行`对应工具
  3. 将结果以 `ToolMessage` 形式喂回模型
  4. LLM 综合原始请求 + 工具结果 → 生成自然语言回答
- ✅ 支持多个工具调用、错误捕获、日志追踪

---

### 🌟 总结：这个脚本的价值
| 特性 | 说明 |
|------|------|
| ✅ **开箱即用的文件读取 Agent** | 用户一句话即可触发读文件 + 解释，无需手动写 `fs.readFile` |
| ✅ **模型无关 & API 兼容** | 通过 `baseURL` 无缝切换 Qwen/Ollama/DeepSeek 等后端 |
| ✅ **标准化工具协议** | 基于 LangChain `tool()` + Zod Schema，可轻松扩展其他工具（如 `run_code`, `search_web`） |
| ✅ **清晰的调试日志** | 每一步（调用、执行、返回）都有 `console.log`，便于排查问题 |




