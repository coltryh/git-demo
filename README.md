# Cloudflare 隧道配置 (GLM-4)
$env:OPENAI_BASE_URL = "https://rapid-brook-ff7d.827238634.workers.dev/api/paas/v4"
$env:OPENAI_API_KEY = "1efd5a531e264686a78cb9af688a4916.zJegTzxa61V0EsIe"

if (!(Test-Path $PROFILE)) { New-Item -Type File -Path $PROFILE -Force }
notepad $PROFILE


# Cloudflare 隧道配置 (GLM-4)
$env:OPENAI_BASE_URL = "https://rapid-brook-ff7d.827238634.workers.dev/api/paas/v4"
$env:OPENAI_API_KEY = "1efd5a531e264686a78cb9af688a4916.zJegTzxa61V0EsIe"

echo $env:OPENAI_BASE_URL
# [新增] 忽略公司防火墙的 SSL 证书拦截
$env:NODE_TLS_REJECT_UNAUTHORIZED = "0"
// proxy.js
const http = require('http');

// ⚠️ 你的 Worker 地址 (请确保没有多余空格)
const WORKER_URL = 'https://rapid-brook-ff7d.827238634.workers.dev'; 

const server = http.createServer(async (req, res) => {
  // 1. 接收 claudecode 发给本地的正常 JSON
  const buffers = [];
  for await (const chunk of req) {
    buffers.push(chunk);
  }
  const rawBody = Buffer.concat(buffers).toString();

  try {
    console.log(`[本地] 收到请求，正在混淆并发送...`);

    // 2. 核心魔法：把 JSON 转成 Base64
    // 使用 Buffer 转换支持中文且更稳定
    const encodedBody = Buffer.from(rawBody).toString('base64');

    // 3. 发送给 Cloudflare Worker
    // 注意：Content-Type 设为 text/plain 伪装成普通文本
    const workerResp = await fetch(WORKER_URL, {
        method: 'POST',
        headers: {
            'Content-Type': 'text/plain' 
        },
        body: encodedBody
    });

    if (!workerResp.ok) {
        throw new Error(`Worker 报错: ${workerResp.status}`);
    }

    // 4. 接收 Worker 返回的混淆响应
    const workerText = await workerResp.text();
    
    // 5. 解码还原 (Base64 -> JSON)
    const decodedBody = Buffer.from(workerText, 'base64').toString('utf-8');

    console.log(`[本地] 收到响应，还原成功，转发给工具`);

    // 6. 像没事人一样还给 claudecode
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(decodedBody);

  } catch (error) {
    console.error('[错误]', error.message);
    res.writeHead(500);
    res.end(JSON.stringify({ error: error.message }));
  }
});

// 监听本地 3000 端口
server.listen(3000, () => {
  console.log('-------------------------------------------');
  console.log('🚀 本地隐形隧道已启动: http://127.0.0.1:3000');
  console.log('🛡️  所有流量将自动加密为 Base64 穿透防火墙');
  console.log('-------------------------------------------');
});




# 指向本地代理 (注意结尾没有 /api/paas/v4，因为 proxy.js 里没处理路径路由，直接转发)
$env:OPENAI_BASE_URL="http://127.0.0.1:3000/v1"

# Key 随便填 (因为 Worker 端已经硬编码了)
$env:OPENAI_API_KEY="any-key"

# 运行
claudecode

// proxy.js
const http = require('http');
// ⚠️ 你的 Worker 地址 (请确保没有多余空格)
const WORKER_URL = 'https://rapid-brook-ff7d.827238634.workers.dev'; 
const server = http.createServer(async (req, res) => {
  // 1. 接收 claudecode 发给本地的正常 JSON
  const buffers = [];
  for await (const chunk of req) {
    buffers.push(chunk);
  }
  const rawBody = Buffer.concat(buffers).toString();
  try {
    console.log(`[本地] 收到请求，正在混淆并发送...`);
    // 2. 核心魔法：把 JSON 转成 Base64
    // 使用 Buffer 转换支持中文且更稳定
    const encodedBody = Buffer.from(rawBody).toString('base64');

    // 3. 发送给 Cloudflare Worker
    // 注意：Content-Type 设为 text/plain 伪装成普通文本
    const workerResp = await fetch(WORKER_URL, {
        method: 'POST',
        headers: {
            'Content-Type': 'text/plain' 
        },
        body: encodedBody
    });
    if (!workerResp.ok) {
        throw new Error(`Worker 报错: ${workerResp.status}`);
    }
    // 4. 接收 Worker 返回的混淆响应
    const workerText = await workerResp.text(); 
    // 5. 解码还原 (Base64 -> JSON)
    const decodedBody = Buffer.from(workerText, 'base64').toString('utf-8')
    console.log(`[本地] 收到响应，还原成功，转发给工具`);

    // 6. 像没事人一样还给 claudecode
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(decodedBody);
  } catch (error) {
    console.error('[错误]', error.message);
    res.writeHead(500);
    res.end(JSON.stringify({ error: error.message }));
  }
});
// 监听本地 3000 端口
server.listen(3000, () => {
  console.log('-------------------------------------------');
  console.log('🚀 本地隐形隧道已启动: http://127.0.0.1:3000');
  console.log('🛡️  所有流量将自动加密为 Base64 穿透防火墙');
  console.log('-------------------------------------------');
});


