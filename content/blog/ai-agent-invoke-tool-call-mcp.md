---
date: '2026-07-10T13:45:25+08:00'
title: 'Ai Agent Invoke Tool Call Mcp'
tags: ['AI Agent', 'Tool Invocation', 'MCP']
---

# MCP 是什么？

一个 `Tool` 的 RPC（远程过程调用）层，用于在 AI Agent 中调用外部工具。
大模型支持 tool calls 就能发现可以使用的 tools，然后让 AI Agent 调用这些 tools 来完成任务。

# mcp `stdio` 示例

## 代码示例
```js
#!/usr/bin/env node
import { readFileSync } from 'node:fs';
import { createInterface } from 'node:readline';

const rl = createInterface({
  input: process.stdin,
  output: process.stdout,
});

rl.on('line', (line) => {
  let req;
  try {
    req = JSON.parse(line);
  } catch {
    return;
  }

  if (req.jsonrpc !== '2.0') return;

  // 1️⃣ initialize
  if (req.method === 'initialize') {
    process.stdout.write(JSON.stringify({
      jsonrpc: '2.0',
      id: req.id,
      result: {
        protocolVersion: '2024-11-05',
        capabilities: {
          tools: {},
        },
        serverInfo: {
          name: 'mcp-readfile-server',
          version: '0.0.1',
        },
      },
    }) + '\n');
    return;
  }

  // 2️⃣ tools/list
  if (req.method === 'tools/list') {
    process.stdout.write(JSON.stringify({
      jsonrpc: '2.0',
      id: req.id,
      result: {
        tools: [
          {
            name: 'read_file',
            description: '读取文件内容。支持相对路径或绝对路径。',
            inputSchema: {
              type: 'object',
              properties: {
                filePath: {
                  type: 'string',
                  description: '要读取的文件路径',
                },
              },
              required: ['filePath'],
            },
          },
        ],
      },
    }) + '\n');
    return;
  }

  // 3️⃣ tools/call
  if (req.method === 'tools/call') {
    const { name, arguments: args } = req.params;

    if (name === 'read_file') {
      try {
        const content = readFileSync(args.filePath, 'utf-8');
        process.stdout.write(JSON.stringify({
          jsonrpc: '2.0',
          id: req.id,
          result: {
            content: [
              {
                type: 'text',
                text: `文件内容:\n${content}`,
              },
            ],
          },
        }) + '\n');
      } catch (err) {
        process.stdout.write(JSON.stringify({
          jsonrpc: '2.0',
          id: req.id,
          result: {
            content: [
              {
                type: 'text',
                text: `读取文件失败: ${err.message}`,
              },
            ],
          },
        }) + '\n');
      }
      return;
    }

    // 未知工具
    process.stdout.write(JSON.stringify({
      jsonrpc: '2.0',
      id: req.id,
      error: {
        code: -32601,
        message: `Unknown tool: ${name}`,
      },
    }) + '\n');
  }
});

```
## 保存为 `mcp-readfile-server.js`，并确保文件具有可执行权限。
## 初始化

```bash
echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{}}' | node mcp-readfile-server.mjs | jq .
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "protocolVersion": "2024-11-05",
    "capabilities": {
      "tools": {}
    },
    "serverInfo": {
      "name": "mcp-readfile-server",
      "version": "0.0.1"
    }
  }
}
```

## 查看工具列表

```bash
echo '{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}' | node mcp-readfile-server.mjs | jq
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "tools": [
      {
        "name": "read_file",
        "description": "读取文件内容。支持相对路径或绝对路径。",
        "inputSchema": {
          "type": "object",
          "properties": {
            "filePath": {
              "type": "string",
              "description": "要读取的文件路径"
            }
          },
          "required": [
            "filePath"
          ]
        }
      }
    ]
  }
}
```

## 调用 read_file 这个工具

```bash
Downloads/mcp echo '{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"read_file","arguments":{"filePath":"/etc/hosts"}}}' | node mcp-readfile-server.mjs | jq
{
  "jsonrpc": "2.0",
  "id": 3,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "文件内容:\n##\n# Host Database\n#\n# localhost is used to configure the loopback interface\n# when the system is booting.  Do not change this entry.\n##\n127.0.0.1\tlocalhost\n255.255.255.255\tbroadcasthost\n::1             localhost\n\n127.0.0.1  mylocalhost\n"
      }
    ]
  }
}
```

# mcp `http` 示例

## 代码示例
```js
#!/usr/bin/env node
import { createServer } from 'node:http';
import { readFileSync } from 'node:fs';
import { randomUUID } from 'node:crypto';

// 存储SSE连接（按sessionId区分客户端）
const sseConnections = new Map();

// 日志写到stderr，避免污染HTTP响应（SSE不能用stdout打日志）
const log = (...args) => process.stderr.write(`[${new Date().toISOString()}] ${args.join(' ')}\n`);

const server = createServer(async (req, res) => {
  // 1. 处理SSE连接（GET /）
  if (req.method === 'GET' && req.url === '/') {
    const sessionId = randomUUID();
    res.writeHead(200, {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      'Connection': 'keep-alive',
      'Access-Control-Allow-Origin': '*', // 开发环境方便测试，生产要关
    });

    // 保存SSE连接
    sseConnections.set(sessionId, res);
    log(`SSE连接建立，sessionId: ${sessionId}`);

    // 第一个SSE事件：告知客户端后续请求的路径
    res.write(`event: endpoint\n`);
    res.write(`data: /message?sessionId=${sessionId}\n\n`);

    // 客户端断开时清理
    req.on('close', () => {
      sseConnections.delete(sessionId);
      log(`SSE连接关闭，sessionId: ${sessionId}`);
    });
    return;
  }

  // 2. 处理JSON-RPC请求（POST /message?sessionId=xxx）
  if (req.method === 'POST' && req.url.startsWith('/message')) {
    const url = new URL(req.url, `http://${req.headers.host}`);
    const sessionId = url.searchParams.get('sessionId');
    const sseRes = sseConnections.get(sessionId);

    if (!sseRes) {
      res.writeHead(404).end('Session not found');
      return;
    }

    // 读取请求体
    let body = '';
    req.on('data', chunk => body += chunk);
    req.on('end', () => {
      let reqJson;
      try {
        reqJson = JSON.parse(body);
      } catch {
        sseRes.write(`event: message\n`);
        sseRes.write(`data: ${JSON.stringify({
          jsonrpc: '2.0',
          id: null,
          error: { code: -32700, message: 'Parse error' }
        })}\n\n`);
        res.writeHead(200).end();
        return;
      }

      log(`收到请求，sessionId: ${sessionId}，method: ${reqJson.method}`);
      const { id, method, params } = reqJson;

      // ===== 以下逻辑和stdio版完全一致 =====
      // 1. initialize
      if (method === 'initialize') {
        sendSseMessage(sseRes, {
          jsonrpc: '2.0',
          id,
          result: {
            protocolVersion: '2024-11-05',
            capabilities: { tools: {} },
            serverInfo: { name: 'mcp-readfile-http', version: '0.0.1' },
          },
        });
      }
      // 2. tools/list
      else if (method === 'tools/list') {
        sendSseMessage(sseRes, {
          jsonrpc: '2.0',
          id,
          result: {
            tools: [{
              name: 'read_file',
              description: '读取文件内容，支持相对/绝对路径',
              inputSchema: {
                type: 'object',
                properties: { filePath: { type: 'string', description: '文件路径' } },
                required: ['filePath'],
              },
            }],
          },
        });
      }
      // 3. tools/call
      else if (method === 'tools/call') {
        const { name, arguments: args } = params;
        if (name === 'read_file') {
          try {
            const content = readFileSync(args.filePath, 'utf-8');
            sendSseMessage(sseRes, {
              jsonrpc: '2.0',
              id,
              result: {
                content: [{ type: 'text', text: `文件内容:\n${content}` }],
              },
            });
          } catch (err) {
            sendSseMessage(sseRes, {
              jsonrpc: '2.0',
              id,
              result: {
                content: [{ type: 'text', text: `读取失败: ${err.message}` }],
              },
            });
          }
        } else {
          sendSseMessage(sseRes, {
            jsonrpc: '2.0',
            id,
            error: { code: -32601, message: `未知工具: ${name}` },
          });
        }
      }
      // 未知方法
      else {
        sendSseMessage(sseRes, {
          jsonrpc: '2.0',
          id,
          error: { code: -32601, message: `未知方法: ${method}` },
        });
      }
      res.writeHead(200).end();
    });
    return;
  }

  // 404其他请求
  res.writeHead(404).end('Not Found');
});

// 封装SSE消息发送
function sendSseMessage(res, data) {
  res.write(`event: message\n`);
  res.write(`data: ${JSON.stringify(data)}\n\n`);
}

// 启动服务
const PORT = 3000;
server.listen(PORT, () => {
  log(`MCP HTTP Server 启动成功：http://localhost:${PORT}`);
});

```

## 保存为 `mcp-readfile-http-server.js`，并确保文件具有可执行权限。
## 启动这个 mcp server

```bash
node mcp-readfile-http-server.js
```

## 访问 http://localhost:3000/ ，浏览器会显示 SSE 连接建立，并返回 sessionId。

```bash
curl -N http://localhost:3000/
event: endpoint
data: /message?sessionId=f2d05540-3384-401a-8a34-cd64c1ecbf33
```

## 初始化

```bash
curl -X POST "http://localhost:3000/message?sessionId=f2d05540-3384-401a-8a34-cd64c1ecbf33" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{}}' | jq
```

## 查看工具列表

```bash
curl -X POST "http://localhost:3000/message?sessionId=f2d05540-3384-401a-8a34-cd64c1ecbf33" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{}}' | jq
```

## 调用 read_file工具

```bash
curl -X POST "http://localhost:3000/message?sessionId=f2d05540-3384-401a-8a34-cd64c1ecbf33" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}' | jq
```


