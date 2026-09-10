=== DeepSeek V4.1-Flash 接入记录 ===
日期: 2026-09-10
机器: 便携版 Hermes (HERMES_HOME = G:\hermes-portable\hermes)

[一、官方事实] (源: api-docs.deepseek.com/quick_start/pricing 实测核对 2026-09-10)
- 永久模型名: deepseek-flash  =  DeepSeek-V4.1-Flash
- 另一模型:   deepseek-v4-pro =  DeepSeek-V4-Pro-0813
  官方将于 2026-09-14 12:00 起逐步退役 Pro，退役后请求全部路由到 V4.1 Flash 并按 Flash 计价
- 已退役但兼容的旧名: deepseek-v4-flash / deepseek-v4-flash-vision-exp / deepseek-chat / deepseek-reasoner
  服务器端仍接受，实际由 V4.1-Flash 服务，按 Flash 价计费
  (实测: 传 deepseek-v4.1-flash 会 400，官方不支持带 4.1 的名字)
- 上下文窗口: 1M tokens  |  最大输出: 384K
- 能力: 思考/非思考双模式、JSON 输出、Tool Calls、Responses API、Anthropic API、
  Chat Prefix Completion、FIM(仅非思考)、Vision 视觉(Flash 支持 / Pro 不支持)
- 计费(每 1M tokens, 美元): 缓存命中输入 0.003(谷)/0.006(峰) | 输入 0.15/0.3 | 输出 0.6/1.2
  峰时 = UTC 周一至周五 01:00-04:00 与 06:00-10:00, 其余时段谷价(半价)
- 并发上限: Flash 2500 | Pro 500
- base_url: https://api.deepseek.com (OpenAI 格式) | https://api.deepseek.com/anthropic (Anthropic 格式)
- 账号余额(2026-09-10): CNY 16.00

[二、本次改动 — 永久链接方案]
1. 主路径固定为"永久组合": provider=deepseek(内置永久 provider 名) + model=deepseek-flash(官方永久别名, 不带版本号)
   → 以后 DeepSeek 再发新版，官方会把 deepseek-flash 指向最新 Flash，配置无需再动
2. custom_providers 条目 deepseek-v4-flash 更名为 deepseek-flash(模型名同步)，备用通道 custom:deepseek-flash
3. auth.json 凭证池键 custom:deepseek-v4-flash → custom:deepseek-flash
4. 源码补丁(2 处, Hermes 升级会覆盖，需要重打):
   (a) hermes-agent/plugins/model-providers/deepseek/__init__.py
       _model_supports_thinking() 增加 deepseek-flash / deepseek-pro 匹配。
       原因: 该函数用 startswith("deepseek-v") 判思考能力，新名字 deepseek-flash 不匹配
       → Hermes 不再显式传 thinking 参数 → DeepSeek 走服务端默认 thinking=on
       → 触发 "reasoning_content must be passed back" HTTP 400 老坑(issue #15700/#17212/#17825)
   (b) hermes-agent/agent/model_metadata.py 增加 "deepseek-flash": 1_000_000
       原因: 否则上下文窗口落到 "deepseek" 兜底值 128K
5. 旧名 deepseek-v4-flash 仍可用(兼容)，改名为的是名正言顺 + 免再改

[三、备份]
- config.yaml / 根 config.yaml / auth.json 各自 .bak_20260910_173220 同目录
- 启动文件快照: G:\HermesBackup\boot_*

[四、回滚]
把 .bak_20260910_173220 覆盖回原文件即可(源码补丁用 git checkout 还原)
