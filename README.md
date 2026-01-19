export const config = {
  runtime: 'edge',
};

export default async function handler(request) {
  if (request.method === 'OPTIONS') {
    return new Response(null, {
      status: 200,
      headers: {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'GET, POST, OPTIONS',
        'Access-Control-Allow-Headers': '*'
      }
    });
  }

  try {
    // 1. 你的智谱 Key
    const API_KEY = "1efd5a531e264686a78cb9af688a4916.zJegTzxa61V0EsIe";

    // 2. 获取 Claude 发来的原始请求
    const body = await request.json();

    // 3. 🚨 关键修改：转发给智谱的 Anthropic 兼容接口
    // 注意：这里必须用 api/anthropic/v1/messages
    const zhipuResponse = await fetch('https://open.bigmodel.cn/api/anthropic/v1/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': API_KEY,      // Anthropic 标准是用 x-api-key
        'anthropic-version': '2023-06-01' // 必须假装是这个版本
      },
      body: JSON.stringify(body)
    });

    // 4. 处理流式响应 (哪怕不流式，原样返回也更稳)
    const data = await zhipuResponse.text();
    
    return new Response(data, {
      headers: { 
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*'
      }
    });

  } catch (e) {
    return new Response(JSON.stringify({ error: e.message }), { 
      status: 500,
      headers: { 'Content-Type': 'application/json' }
    });
  }
}





const http = require('http');

process.env.NODE_TLS_REJECT_UNAUTHORIZED = '0';

// 你的 Vercel 地址
const VERCEL_URL = 'https://api.ryhcolt.online/api'; 
// 强制替换的模型
const FORCE_MODEL = 'glm-4.7'; 

const server = http.createServer(async (req, res) => {
    // CORS 头
    res.setHeader('Access-Control-Allow-Origin', '*');
    res.setHeader('Access-Control-Allow-Headers', '*');

    if (req.method === 'OPTIONS') { res.writeHead(200); res.end(); return; }

    if (req.method === 'POST') {
        let body = '';
        req.on('data', chunk => body += chunk);
        req.on('end', async () => {
            try {
                const originalRequest = JSON.parse(body);
                
                // 强制关闭 stream (为了兼容性，先让它一次性返回，解决“不说话”的问题)
                // 智谱的 Anthropic 接口对流式支持比较复杂，先用非流式跑通
                originalRequest.stream = false; 
                originalRequest.model = FORCE_MODEL;

                console.log(`🔌 请求: ${FORCE_MODEL} | 模式: 极速响应`);

                const vercelResp = await fetch(VERCEL_URL, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(originalRequest)
                });

                const data = await vercelResp.text();
                
                // 打印一下智谱到底返回了什么，方便调试
                console.log("📦 智谱返回数据长度:", data.length);
                if (data.length < 500) console.log("🔍 内容预览:", data);

                res.writeHead(vercelResp.status, { 'Content-Type': 'application/json' });
                res.end(data);

            } catch (error) {
                console.error('❌ 错误:', error.message);
                res.writeHead(500);
                res.end(JSON.stringify({ error: error.message }));
            }
        });
    }
});

server.listen(3000, () => {
    console.log('🚀 增强版代理已启动 (适配 Anthropic 协议)');
});

