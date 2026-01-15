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
