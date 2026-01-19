const http = require('http');

// 1. 设置 SSL 忽略
process.env.NODE_TLS_REJECT_UNAUTHORIZED = '0';

const VERCEL_URL = 'https://api.ryhcolt.online/api';
const FORCE_MODEL = 'glm-4.7';

const server = http.createServer(async (req, res) => {
    // CORS 配置
    res.setHeader('Access-Control-Allow-Origin', '*');
    res.setHeader('Access-Control-Allow-Methods', 'POST, OPTIONS');
    res.setHeader('Access-Control-Allow-Headers', '*');

    if (req.method === 'OPTIONS') {
        res.writeHead(200);
        res.end();
        return;
    }

    if (req.method === 'POST') {
        let body = '';
        req.on('data', chunk => body += chunk);
        req.on('end', async () => {
            try {
                const originalRequest = JSON.parse(body);
                console.log(`🔌 拦截请求: ${originalRequest.model} -> ⚡️ 换成: ${FORCE_MODEL}`);
                originalRequest.model = FORCE_MODEL;

                const vercelResp = await fetch(VERCEL_URL, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'User-Agent': 'curl/7.68.0'
                    },
                    body: JSON.stringify(originalRequest)
                });

                if (!vercelResp.ok) {
                    const errText = await vercelResp.text();
                    console.error(`❌ Vercel 报错: ${vercelResp.status}`, errText);
                    res.writeHead(vercelResp.status, { 'Content-Type': 'application/json' });
                    res.end(JSON.stringify({ error: errText }));
                    return;
                }

                const data = await vercelResp.json();
                console.log(`✅ 成功响应`);
                res.writeHead(200, { 'Content-Type': 'application/json' });
                res.end(JSON.stringify(data));

            } catch (error) {
                console.error('❌ 内部错误:', error.message);
                res.writeHead(500);
                res.end(JSON.stringify({ error: error.message }));
            }
        });
    } else {
        res.writeHead(404);
        res.end('Not Found');
    }
});

// 🔥 新增：错误捕获逻辑
server.on('error', (e) => {
    if (e.code === 'EADDRINUSE') {
        console.error('❌ 启动失败！！端口 3000 已经被占用了。');
        console.error('👉 请先运行 taskkill /F /IM node.exe 关闭旧进程，或者修改代码里的端口号。');
    } else {
        console.error('❌ 发生未知错误:', e);
    }
});

// 尝试启动
server.listen(3000, () => {
    console.log('------------------------------------------------');
    console.log('🚀 本地代理已成功启动！(不要关闭这个窗口)');
    console.log(`🤖 目标模型: ${FORCE_MODEL}`);
    console.log('📡 监听地址: http://127.0.0.1:3000');
    console.log('------------------------------------------------');
});
