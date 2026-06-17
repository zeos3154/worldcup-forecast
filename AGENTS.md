模型使用方式在README.md，先阅读它
ODDS_API_KEY放在项目.env下

# 模型预测


#sporttery采集规则

使用chrome devtools mcp调用浏览器采集，不要使用firecrawl会被拦截
具体url为：
- https://m.sporttery.cn/mjc/jsq/zqspf/
- https://m.sporttery.cn/mjc/jsq/zqbf/
- https://m.sporttery.cn/mjc/jsq/zqzjq/
- https://m.sporttery.cn/mjc/jsq/zqbqc/
- https://m.sporttery.cn/mjc/jsq/zqhhgg/
获取当前时间，采集今日赛程
根据今日赛程采集数据全量写入sporttery/ 目录，根据比赛时间按照`MM-DD.md`格式存档，按照如下模板存档
```markdown
# 01_胜平负

# 02_比分

# 03_总进球

# 04_半全场
```

# 模型预测使用

# 注意
- 左上角有三角的“单”表示可以单独购买胜平负，否则只能走混合过关
- 明确要求“小博大”或 2 元混 4 关 娱乐票

python虚拟环境在.venv/，必须使用该虚拟环境运行模型

