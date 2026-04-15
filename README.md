🎯 AI Job Hunter
English | 中文

Too many job listings, don't know where to apply? Tell Claude your background and target company — it will automatically scrape the official careers page, read each job's full details, and generate an Excel sheet with personalized match ratings. Works for any company and any role. Best used with Claude Cowork.

What It Does
Automatically visits company career pages and searches by keyword
Reads every job's full detail page (responsibilities + requirements), not just list summaries
Rates each job based on your personal background: 🟢 Strong Match / 🟡 Possible / 🔴 Not a Fit
Outputs a color-coded Excel sheet with match notes and direct application links
Requirements
Environment	Support	Notes
Claude Cowork	✅ Full	Recommended — browser agent bypasses anti-scraping
Claude Code	⚠️ Limited	Major company sites may block access
Claude.ai Web	⚠️ Limited	Same as above
Installation
⭐ Option 1: Install .skill file (Recommended)
Easiest — no technical setup required.

Go to Releases and download ai-job-hunter.skill
Open Claude Cowork
Drag the .skill file into the chat, or upload via Settings → Install Skill
Option 2: Clone the repo
bash
git clone https://github.com/tianhaoyang778-hue/ai-job-hunter.git
Place the ai-job-hunter/ folder in your Claude skills directory: ~/.claude/skills/user/ai-job-hunter/

Option 3: SKILL.md only (Minimal)
Download SKILL.md, create a folder ai-job-hunter/, and place it inside.

⚠️ This skips the Excel script and built-in parser logic. Not recommended for regular use.

How to Use
After installation, just say in Claude Cowork:

Find me AI product internships at ByteDance
Does Tencent have any operations internships?
Find internships suitable for a liberal arts background
Make me a spreadsheet of data analyst roles at Alibaba
Claude will trigger the skill automatically, ask for your keywords and background, then start running.

Built-in Company Parsers
Company	Status	Last Verified
ByteDance (字节跳动)	✅ Built-in	2026-04-14
Other companies	🔄 Auto-learned on first run	—
How Ratings Work
Ratings are based entirely on the background you describe — no fixed template. The skill includes a real user's background as a formatting example (prompts/scoring/example-mine.md). Claude uses that structure to generate a custom rating standard for you each time.

FAQ
Q: Why must I use Cowork? Major career sites have anti-scraping protection. Cowork's browser agent simulates real browsing and gets through. Other environments fall back to search engine results, which are incomplete.

Q: Do I need to log in to the career site? Most job detail pages are public. If login is required, Claude will pause and ask you to log in before continuing.

Q: What happens to jobs that fail to scrape? Claude lists all failed jobs with their links at the end. You can open them manually and paste the content back to Claude to add to the spreadsheet.

Project Structure
ai-job-hunter/
├── SKILL.md                          # Main instruction file for Claude
├── README.md                         # This file
├── LICENSE
├── agents/
│   └── detail-scraper.md             # Job detail page scraping agent
├── parsers/
│   ├── README.md
│   └── bytedance.md                  # ByteDance scraping logic
├── prompts/
│   └── scoring/
│       ├── README.md
│       └── example-mine.md           # Rating prompt example (real background)
├── references/
│   ├── company_urls.md               # Career page URLs reference
│   └── session-example-bytedance.md  # Full ByteDance session record
├── templates/
│   └── 字节跳动AI岗位_推荐_.xlsx      # Excel style reference
└── scripts/
    └── generate_excel.py             # Excel generation script
Contributing
PRs welcome! To add a new company parser, create parsers/company-name.md following the format in parsers/bytedance.md.

License
MIT

中文
English | 中文

岗位太多不知道该投哪个？告诉 Claude 你的背景和目标公司，自动抓取官网岗位、逐一读取详情页原文，生成带个性化匹配评级的 Excel 表格。支持任意公司、任意岗位方向。推荐在 Claude Cowork 中使用。

能做什么
自动访问大厂招聘官网，按关键词搜索岗位
逐一读取每个岗位的详情页原文（职责 + 要求），不依赖列表页摘要
根据你的个人背景对每个岗位评级：🟢 高 / 🟡 中 / 🔴 低
输出带颜色评级、匹配说明、详情页链接的 Excel 表格
环境要求
环境	支持程度	说明
Claude Cowork	✅ 完整功能	推荐，浏览器 Agent 可绕过反爬
Claude Code	⚠️ 功能受限	大厂官网可能无法访问
Claude.ai 网页版	⚠️ 功能受限	同上
安装方法
⭐ 方法一：下载 .skill 文件安装（推荐）
最简单，无需任何技术背景。

前往 Releases 页面下载 ai-job-hunter.skill
打开 Claude Cowork
将 .skill 文件拖入对话框，或通过设置页面「安装 Skill」上传
方法二：克隆仓库手动安装
bash
git clone https://github.com/tianhaoyang778-hue/ai-job-hunter.git
将 ai-job-hunter/ 文件夹放入 Claude skills 目录： ~/.claude/skills/user/ai-job-hunter/

方法三：只下载 SKILL.md（最简安装）
下载 SKILL.md，新建 ai-job-hunter/ 文件夹，将文件放入其中。

⚠️ 此方式缺少 Excel 生成脚本和内置爬取逻辑，不推荐长期使用。

使用方式
安装后在 Claude Cowork 中直接说：

帮我搜索字节跳动的AI产品实习
腾讯有没有运营方向的实习？
给我找适合文科背景的互联网实习
整理一份阿里巴巴的数据分析岗位表格
Claude 会自动触发 skill，引导你填写关键词和个人背景后开始执行。

已内置的公司爬取逻辑
公司	状态	最后验证
字节跳动	✅ 已内置	2026-04-14
其他公司	🔄 首次运行自动学习	—
评级逻辑说明
评级完全基于你描述的个人背景，没有固定标准，适用于任何行业和方向。skill 内置了一份真实用户背景的评级示范（prompts/scoring/example-mine.md），Claude 每次参考其结构，根据你的背景重新生成专属评级标准。

常见问题
Q：为什么一定要用 Cowork？ 大厂招聘官网有反爬保护，只有 Cowork 的浏览器 Agent 能稳定访问。其他环境会降级为搜索引擎结果，数据不完整。

Q：需要登录招聘官网吗？ 大部分详情页不需要登录。遇到需要登录的情况，Claude 会暂停提示你完成登录后继续。

Q：抓取失败的岗位怎么处理？ Claude 会在生成 Excel 前列出所有失败的岗位和链接，你可以手动打开后将内容复制给 Claude 补充进表格。

项目结构
ai-job-hunter/
├── SKILL.md                          # 主控文件，Claude 执行逻辑
├── README.md                         # 本文件
├── LICENSE
├── agents/
│   └── detail-scraper.md             # 详情页抓取 agent 指令
├── parsers/
│   ├── README.md
│   └── bytedance.md                  # 字节跳动爬取逻辑
├── prompts/
│   └── scoring/
│       ├── README.md
│       └── example-mine.md           # 评级 prompt 示范（真实背景）
├── references/
│   ├── company_urls.md               # 主流公司招聘 URL 速查
│   └── session-example-bytedance.md  # 字节跳动完整实战记录
├── templates/
│   └── 字节跳动AI岗位_推荐_.xlsx      # Excel 样式参考
└── scripts/
    └── generate_excel.py             # Excel 生成脚本
Contributing
欢迎提交 PR 补充新公司的爬取逻辑，格式参考 parsers/bytedance.md。

License
MIT
