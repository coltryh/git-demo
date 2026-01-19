const http = require('http');

// 1. 忽略证书错误 (公司内网防拦截)
process.env.NODE_TLS_REJECT_UNAUTHORIZED = '0';

// 2. 你的 Vercel 地址
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
                
                // 🔥 关键修改 1：开启流式传输 (Stream)
                // 只要你是 Vercel Edge Runtime，流式传输就不会超时
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

                // 🔥 关键修改 2：管道式转发 (Pipe)
                // 不等全部结果，收到一点就转发一点，保持连接活跃
                res.writeHead(200, {
                    'Content-Type': 'text/event-stream',
                    'Cache-Control': 'no-cache',
                    'Connection': 'keep-alive'
                });

                const reader = vercelResp.body.getReader();
                const decoder = new TextDecoder();

                while (true) {
                    const { done, value } = await reader.read();
                    if (done) break;
                    // 将收到的二进制流直接转发给 Claude Code
                    res.write(value);
                }
                res.end();
                console.log("✅ 流式传输完成");

            } catch (error) {
                console.error('❌ 代理报错:', error.message);
                // 如果头还没发，发个 500；如果发了，就直接断开
                if (!res.headersSent) {
                    res.writeHead(500);
                    res.end(JSON.stringify({ error: error.message }));
                } else {
                    res.end();
                }
            }
        });
    }
});

server.listen(3000, () => {
    console.log('-------------------------------------------');
    console.log('🚀 防超时流式基站已启动！(端口: 3000)');
    console.log('📡 模式: Stream = True');
    console.log('-------------------------------------------');
});
