# ggAPP框架解读

## functions.lua

基础全局函数 `gg.base.util.functions`

1. skynet.getenv(key) 扩展skynet环境变量
    * 优先加载 `app.config.custom` 中的配置,并且其中的配置会覆盖*.config
