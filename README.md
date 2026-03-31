claude mcp add -s user MiniMax --env MINIMAX_API_KEY=api_key --env MINIMAX_API_HOST=https://api.minimaxi.com -- uvx minimax-coding-plan-mcp -y

sk-cp-ci7wMCIWzMmkymTp0VdexCloEVWjevQZ-OqJzHzpcMPfYMPbRWHUzP50_QbSREsD7UTszpw4O1fEMU8T2-qaORrvGdnr7f-La3dJ7Qd7uw85sxgk349JAl0

powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 |


 iex"



{
  "mcpServers": {
    "MiniMax": {
      "command": "uvx",
      "args": ["minimax-coding-plan-mcp", "-y"],
      "env": {
        "MINIMAX_API_KEY": "MINIMAX_API_KEY",
        "MINIMAX_API_HOST": "https://api.minimaxi.com"
      }
    }
  }
}

(Get-Command uvx).source

# 1. 强制忽略所有的 SSL 证书报错
[Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}

# 2. 重新执行安装命令
irm https://astral.sh/uv/install.ps1 | iex
