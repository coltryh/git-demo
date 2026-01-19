export const config = {
  runtime: 'edge', // 必须使用 Edge 运行时，它支持流式传输且超时时间更长
};

export default async function handler(request) {
  // 1. 处理 CORS 预检
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
    const API_KEY = "1efd5a531e264686a78cb9af688a4916.zJegTzxa61V0EsIe"; // 你的 Key

    const body = await request.json();

    // 2. 强制开启流式 (Stream)
    // 无论客户端发什么，我们都强制要求智谱流式返回
    body.stream = true;

    // 3. 发送给智谱 (Anthropic 兼容接口)
    const zhipuResponse = await fetch('https://open.bigmodel.cn/api/anthropic/v1/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': API_KEY,
        'anthropic-version': '2023-06-01'
      },
      body: JSON.stringify(body)
    });

    // 4. 🔥 核心修改：直接透传 (Pipe)
    // 不要用 await response.text()！直接把 body 塞回去！
    // 这样智谱每吐一个字，你那边就能立马收到，Vercel 就不会 504 了。
    return new Response(zhipuResponse.body, {
      status: zhipuResponse.status,
      headers: {
        'Content-Type': 'text/event-stream', // 声明这是流
        'Access-Control-Allow-Origin': '*',
        'Connection': 'keep-alive'
      }
    });

  } catch (e) {
    return new Response(JSON.stringify({ error: e.message }), { status: 500 });
  }
}
