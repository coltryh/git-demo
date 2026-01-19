$settingsPath = "$env:USERPROFILE\.claude\settings.json"
$settingsContent = @"
{
  "env": {
    "ANTHROPIC_BASE_URL": "http://127.0.0.1:3000/v1",
    "ANTHROPIC_AUTH_TOKEN": "any",
    "API_TIMEOUT_MS": "3000000",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1",
    "NODE_TLS_REJECT_UNAUTHORIZED": "0"
  }
}
"@
Set-Content -Path $settingsPath -Value $settingsContent
Write-Host "✅ 配置已更新！现在指向本地代理 http://127.0.0.1:3000" -ForegroundColor Green
