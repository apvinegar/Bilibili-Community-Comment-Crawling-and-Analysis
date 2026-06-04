# BCFC — Bilibili Community Friendliness Crawler

🎮 **B站二次元手游社区友好度分析系统**

爬取 B站 热门二次元手游相关视频的评论和弹幕，使用 LLM 进行多维度情感分析，用机器学习方法评价各游戏社区的友好程度。

## 支持的游戏

原神、鸣潮、明日方舟、终末地、异环、崩坏3、星穹铁道、绝区零、幻塔、碧蓝航线、少女前线、重返未来：1999、蔚蓝档案、FGO、阴阳师

## 安装

```bash
# 克隆项目
cd Bill

# 安装依赖
pip install -e .

# 可选：安装可视化依赖
pip install -e ".[viz]"
```

## 配置

1. 复制环境变量模板：
```bash
cp .env.example .env
```

2. 编辑 `.env`，填入你的 B站 Cookie 和 LLM API Key：
```
BILIBILI_SESSDATA=你的sessdata
BILIBILI_BILI_JCT=你的bili_jct
DEEPSEEK_API_KEY=你的deepseek_api_key
```

3. 自定义 `config.yaml`（可选）：调整爬取参数、LLM 模型、评分权重等。

## 使用

### 1. 初始化数据库

```bash
bcfc init
```

### 2. 爬取数据

```bash
# 爬取单个游戏
bcfc crawl --game "原神" --max-videos 100

# 爬取所有游戏
bcfc crawl --all-games --max-videos 300

# 断点续传
bcfc crawl --resume
```

### 3. LLM 分析

```bash
# 分析所有未分析文本
bcfc analyze --all-unanalyzed

# 仅分析评论
bcfc analyze --type comments --max-texts 5000
```

### 4. 计算社区友好度

```bash
# 按季度计算
bcfc score --all-games --quarterly

# 全期汇总
bcfc score --all-games --no-quarterly
```

### 5. 生成报告

```bash
# Markdown 报告
bcfc report --format markdown

# CSV 导出
bcfc report --format csv

# JSON 导出
bcfc report --format json

# 可视化图表
bcfc report --format chart

# 交互式 HTML 仪表板
bcfc report --format interactive

# 所有格式
bcfc report --format all
```

### 6. 一键完整流程

```bash
bcfc run --all-games --max-videos 200 --report-format all
```

## 工作原理

### 数据爬取
1. 通过 B站搜索 API 按关键词查找视频（2018-2026）
2. 获取评论（游标分页 + 子评论）和弹幕（实时 Protobuf + 历史 XML）
3. 使用 WBI 签名和 Cookie 认证，SQLite 存储

### LLM 分析
五维度评分体系（0-100 分）：
- **友好度 (friendliness)**: 对他人是否友善包容
- **毒性 (toxicity)**: 是否含有恶意、攻击性言论
- **建设性 (constructiveness)**: 讨论的质量和贡献度
- **情感倾向 (sentiment)**: 对游戏/内容的正面程度
- **引战度 (controversy)**: 是否容易引发争论对立

包含 B站特有表达的识别（"典""乐""急""孝"等）和规则预过滤。

### CFI 评分公式

```
CFI = 0.25×(100-毒性) + 0.25×友好度 + 0.15×建设性
    + 0.15×(情感-50)×2 - 0.10×引战度
    + 0.05×高质量比率×100 + 0.05×(1-高毒性比率)×100
```

CFI 归一化到 0-100，越高表示社区越友好。

## 项目结构

```
Bill/
├── src/bcfc/           # 主包
│   ├── crawler/        # B站数据爬取
│   ├── db/             # SQLite 数据库
│   ├── analysis/       # LLM 分析
│   ├── ml/             # ML 评分
│   ├── report/         # 导出和可视化
│   └── main.py         # CLI 入口
├── config.yaml         # 配置文件
├── pyproject.toml      # 项目元数据
└── data/               # 数据输出
```

## 注意事项

- 爬取历史弹幕需要 B站登录 Cookie（SESSDATA）
- 请遵守 B站使用条款，合理控制请求频率
- 建议先对少量视频测试，确认配置正确后再大规模运行
- LLM 分析会产生 API 费用，建议先使用 `--max-texts` 限制数量

## License

MIT
