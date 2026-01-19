const http = require('http');

// 1. 忽略证书错误 (公司内网防拦截)
process.env.NODE_TLS_REJECT_UNAUTHORIZED = '0';

// 2. 你的 Vercel 地址 (确保这个地址是你部署好的)
const VERCEL_URL = 'https://api.ryhcolt.online/api'; 
// 3. 强制替换的模型
const FORCE_MODEL = 'glm-4.7'; 

const server = http.createServer(async (req, res) => {
    // 设置 CORS 头
    res.setHeader('Access-Control-Allow-Origin', '*');
    res.setHeader('Access-Control-Allow-Headers', '*');

    // 处理预检
    if (req.method === 'OPTIONS') { res.writeHead(200); res.end(); return; }

    if (req.method === 'POST') {
        let body = '';
        req.on('data', chunk => body += chunk);
        req.on('end', async () => {
            try {
                const originalRequest = JSON.parse(body);
                
                // 🔥 关键配置：开启流式 + 强制模型
                originalRequest.stream = true; 
                originalRequest.model = FORCE_MODEL;

                console.log(`🔌 收到请求 -> 🚀 转发流式请求 (${FORCE_MODEL})`);

                const vercelResp = await fetch(VERCEL_URL, {
                    method: 'POST',
                    headers: { 
                        'Content-Type': 'application/json',
                        'User-Agent': 'curl/7.68.0' 
                    },
                    body: JSON.stringify(originalRequest)
                });

                // 处理 Vercel 报错
                if (!vercelResp.ok) {
                    const errText = await vercelResp.text();
                    console.error(`❌ Vercel 报错: ${vercelResp.status}`, errText);
                    res.writeHead(vercelResp.status, { 'Content-Type': 'application/json' });
                    res.end(JSON.stringify({ error: errText }));
                    return;
                }

                // 🔥 管道式转发 (Pipe)
                res.writeHead(200, {
                    'Content-Type': 'text/event-stream',
                    'Cache-Control': 'no-cache',
                    'Connection': 'keep-alive'
                });

                const reader = vercelResp.body.getReader();
                while (true) {
                    const { done, value } = await reader.read();
                    if (done) break;
                    res.write(value);
                }
                res.end();
                console.log("✅ 流式传输完成");

            } catch (error) {
                console.error('❌ 代理报错:', error.message);
                if (!res.headersSent) {
                    res.writeHead(500);
                    res.end(JSON.stringify({ error: error.message }));
                }
            }
        });
    }
});

server.listen(3000, () => {
    console.log('-------------------------------------------');
    console.log('🚀 本地流式基站已启动！(端口: 3000)');
    console.log('📡 随时准备连接...');
    console.log('-------------------------------------------');
});
