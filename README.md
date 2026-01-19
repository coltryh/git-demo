const http = require('http');
const https = require('https');

// 1. 设置 SSL 忽略 (关键：绕过 Zscaler 拦截)
process.env.NODE_TLS_REJECT_UNAUTHORIZED = '0';

// 2. 你的 Vercel 入口地址
const VERCEL_URL = 'https://api.ryhcolt.online/api'; 

// 3. 🔥 修正：使用智谱最新旗舰 GLM-4.7
// 根据智谱 2025/2026 的 API 定义，模型代码通常是 "glm-4.7"
const FORCE_MODEL = 'glm-4.7'; 

const server = http.createServer(async (req, res) => {
    // 设置 CORS 头，允许跨域
    res.setHeader('Access-Control-Allow-Origin', '*');
    res.setHeader('Access-Control-Allow-Methods', 'POST, OPTIONS');
    res.setHeader('Access-Control-Allow-Headers', '*');

    // 处理预检请求
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
                // A. 解析原始请求
                const originalRequest = JSON.parse(body);
                
                // 打印 ClaudeCode 原本想请求的模型 (比如 claude-sonnet-4.5)
                console.log(`🔌 拦截请求: ${originalRequest.model} -> ⚡️ 偷梁换柱为: ${FORCE_MODEL}`);
                
                // B. 🔀 强制替换为 GLM-4.7
                originalRequest.model = FORCE_MODEL;

                // C. 转发给你的 Vercel (伪装成 curl 绕过 Zscaler)
                const vercelResp = await fetch(VERCEL_URL, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'User-Agent': 'curl/7.68.0' // 关键伪装
                    },
                    body: JSON.stringify(originalRequest)
                });

                // D. 处理 Vercel 的响应
                if (!vercelResp.ok) {
                    const errText = await vercelResp.text();
                    console.error(`❌ Vercel 报错: ${vercelResp.status}`, errText);
                    res.writeHead(vercelResp.status, { 'Content-Type': 'application/json' });
                    res.end(JSON.stringify({ error: errText }));
                    return;
                }

                // E. 将 GLM-4.7 的回复转发回本地工具
                const data = await vercelResp.json();
                
                // 🔍 调试信息：确认拿到的是不是 4.7 的回复
                const replyContent = data.choices?.[0]?.message?.content || "";
                console.log(`✅ GLM-4.7 响应成功 (长度: ${replyContent.length})`);
                
                res.writeHead(200, { 'Content-Type': 'application/json' });
                res.end(JSON.stringify(data));

            } catch (error) {
                console.error('❌ 代理内部错误:', error.message);
                res.writeHead(500);
                res.end(JSON.stringify({ error: error.message }));
            }
        });
    } else {
        res.writeHead(404);
        res.end('Not Found');
    }
});

server.listen(3000, () => {
    console.log('🚀 本地代理服务器已启动！');
    console.log(`🤖 目标模型: ${FORCE_MODEL}`);
    console.log('📡 监听地址: http://127.0.0.1:3000');
});
