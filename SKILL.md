---
name: ai-job-hunter
description: >
  招聘岗位自动搜索与Excel输出工具。只要用户提到以下任意意图，立刻使用此skill，无需用户明确说"skill"：
  「帮我搜索/找/查 [公司] 的实习」「[公司] 有没有[岗位类型]实习」「给我找适合[背景描述]的岗位」
  「整理一份 [公司] 的实习表格」「帮我看看 [公司] 在招什么岗位」「我想投[公司]的实习」。
  Skill会自动完成：检测运行环境 → 询问用户岗位需求和个人背景 → 浏览器抓取岗位列表 → 逐一读取每个岗位详情页原文 → 根据用户背景评级 → 生成带颜色评级的Excel表格。
  适用于任意公司、任意岗位类型，不限制行业或背景。
compatibility: 推荐在 Claude Cowork 中运行（需要浏览器Agent）。Claude Code 和 Claude.ai 因网络限制，大厂官网可能无法访问。
---

# 招聘岗位搜索 Skill

## 重要说明

**本 skill 推荐在 Claude Cowork 中运行。** 大厂招聘官网有反爬保护，只有 Cowork 的浏览器 Agent 能稳定访问。Claude Code 和 Claude.ai 网页版遇到被拦截时会自动降级为搜索引擎结果，数据完整性无法保证。

---

## 工作流总览

```
Step 0  环境检测与说明
Step 1  收集用户需求（岗位关键词 + 个人背景 + 运行模式）
Step 2  获取招聘URL（内置列表 → URL验活 → 自动重查）
Step 3  浏览器打开页面 + 筛选设置
Step 4  获取岗位列表 + 翻页策略确认
Step 5  逐一抓取每个岗位详情页（强制，含数据持久化）
Step 6  根据用户背景评级
Step 7  生成Excel输出
```

每步有明确的降级路径，遇到失败不中断，按降级顺序继续。

---

## Step 0：环境检测与说明

检测当前环境，规则如下：

- 检测到 **浏览器MCP工具**（`navigate`、`find`、`computer` 等）→ **Cowork模式** ✅ 完整功能
- 有 bash_tool，无浏览器MCP → **Claude Code模式** ⚠️ 功能受限
- 无 bash_tool → **Web模式** ⚠️ 功能受限

**若为 Cowork 模式**，直接进入 Step 1。

**若为 Claude Code 或 Web 模式**，告知用户：
> 「当前环境（Claude Code / Web）对大厂招聘官网的访问受到网络限制，可能无法获取完整岗位信息。建议切换到 Claude Cowork 使用本 skill 以获得最佳效果。是否仍要继续？」

用户确认继续则执行，降级策略见各步骤末尾。

---

## Step 1：收集用户需求

**一次性询问所有信息，不分开问：**

### 基本参数

| 参数 | 说明 |
|------|------|
| 公司名 | 必填，如：字节跳动、腾讯、阿里 |
| 岗位关键词 | 必填，可多个，见下方提示 |
| 目标城市 | 选填，默认不限 |
| 实习届别 | 选填，默认日常实习 |

**关键词提示**（不同公司命名不同，建议多填）：
> AI相关：「AI产品、AI运营、AIGC运营、智能产品、大模型产品」
> 数据方向：「数据分析、数据运营、业务分析」
> 内容方向：「内容运营、内容策划、创作者运营」
> 不确定时：描述岗位方向，由 Claude 推荐关键词

### 个人背景描述

请用自然语言描述特长和优势，越具体越准确：
> 「例如：传播学背景，熟练使用 ChatGPT/Midjourney 等AI工具，做过公众号运营（10w+），不想写代码，不做纯数据标注」

### 运行模式

- **快速模式**：只抓前5页，验证流程，快速出结果
- **细致模式**：分批处理，每5页预览确认，最终合并完整Excel

---

## Step 2：获取招聘URL

### 2a. 查找URL

**先检查 `parsers/` 文件夹**，若有目标公司对应的文件（如 `parsers/bytedance.md`），直接读取其中的 URL 模板和爬取说明，跳过 2b 中的摸索步骤。

若 `parsers/` 无对应文件，参考 `references/company_urls.md` 获取目标公司的招聘页面URL。

若两处都没有，用 web_search 搜索：「[公司名] 校园招聘官网 实习 [当前年份]」

### 2b. URL验活（必须执行）

拿到URL后，**先用浏览器访问验证是否有效**，判断标准：

| 结果 | 判断 | 处理 |
|------|------|------|
| 正常加载招聘页面 | ✅ 有效 | 继续 Step 3 |
| 404 / 空白页 | ❌ 过期 | 执行 2c 重查 |
| 跳转到官网首页 | ❌ 过期 | 执行 2c 重查 |
| 跳转到登录页 | ⚠️ 需登录 | 提示用户在浏览器中登录后继续 |

### 2c. URL过期时自动重查

```
1. web_search：「[公司名] 校园招聘 实习 官网 [当前年份]」
2. 从搜索结果中识别官方域名（排除第三方招聘平台）
3. 用浏览器访问候选URL，重新验活
4. 找到有效URL后更新，继续执行
5. 若3次重查仍失败，告知用户并提供最后找到的官网链接，请用户手动确认
```

### 2d. 告知用户

确认URL有效后告知：
> 「即将访问 [公司名] 官方招聘页面（[URL]）。页面为公开内容，如需查看详情页需要账号登录，建议提前在浏览器中完成登录。是否继续？」

---

## Step 3：浏览器打开页面 + 筛选设置

### 3a. 导航到招聘页面

用浏览器工具导航到招聘URL，等待页面完全加载。

### 3b. 设置筛选条件

按以下顺序操作筛选：

**① 职位类别筛选（如有）**

找到职位类别筛选区域，勾选与用户关键词相关的分类（如"产品"、"运营"）：
```
1. find 工具定位筛选区域
2. 识别可勾选的分类项
3. computer 工具点击对应分类
4. 等待页面响应
```

**② 搜索框输入关键词**

⚠️ 搜索框操作必须按以下顺序，禁止用键盘模拟输入：

```
方法A（推荐）：
1. find 工具定位搜索框，获取元素引用ID
2. form_input 工具传入元素ID和关键词
3. find 工具定位搜索/提交按钮
4. computer 工具点击按钮
5. 等待结果加载

方法B（form_input 失败时）：
1. javascript_tool 直接设置值：
   document.querySelector('input[type="search"],input[placeholder*="搜索"],input[placeholder*="职位"]').value = '关键词'
2. 触发 input 事件：
   el.dispatchEvent(new Event('input', {bubbles: true}))
3. computer 工具点击搜索按钮

方法C（以上均失败时）：
直接在URL中添加关键词参数导航（参考 references/company_urls.md 中的URL模板）
```

**③ 城市筛选（如有且用户指定了城市）**

同上，用 find + computer 选择城市。

---

## Step 4：获取岗位列表 + 翻页策略

### 4a. 读取当前页岗位列表

用 `get_page_text` 或 `read_page` 读取页面内容，提取：
- 岗位名称
- 详情页链接
- 岗位地点（如列表页有显示）

若直接读取失败，用 javascript_tool 提取：
```javascript
const cards = document.querySelectorAll('[class*="position"],[class*="job"],[class*="item"]');
const jobs = [];
cards.forEach(el => {
  const link = el.querySelector('a');
  jobs.push({
    title: el.querySelector('[class*="title"],[class*="name"]')?.innerText?.trim(),
    url: link?.href,
    location: el.querySelector('[class*="location"],[class*="city"]')?.innerText?.trim()
  });
});
return JSON.stringify(jobs.filter(j => j.title && j.url));
```

### 4b. 获取总页数

读取分页区域，识别总页数。识别方式：
```
优先：读取分页组件文字（如"共123页"、"1/123"）
次选：javascript_tool 提取分页数据
      document.querySelector('[class*="pagination"],[class*="pager"]')?.innerText
```

### 4c. 翻页策略确认

**总页数 ≤ 5页**：直接全部抓取，无需询问

**总页数 > 5页**：
> 「共找到 [N] 页结果。您选择了[快速/细致]模式：
> - 快速模式：只处理前5页，快速出结果
> - 细致模式：每5页一批，预览确认后继续，最终合并完整Excel
> 确认继续？」

### 4d. 翻页执行

每页抓取完后翻到下一页，按以下顺序尝试：

```
方法1（URL参数翻页）：
  直接修改URL中的page/current参数导航到下一页
  等待加载，验证内容已更新（检查第一个岗位名称是否变化）

方法2（点击翻页按钮）：
  find 工具定位"下一页"按钮
  computer 工具点击
  等待加载，用 get_page_text 验证内容已更新

方法3（JS注入翻页）：
  javascript_tool 触发翻页：
  document.querySelector('[class*="next"],[aria-label*="next"],[aria-label*="下一页"]')?.click()
  等待500ms后读取内容

验证翻页成功：
  对比翻页前后第一个岗位的标题，若相同则翻页失败，重试其他方法
  连续3次失败则停止，告知用户当前已抓取页数，询问是否生成现有数据的Excel
```

---

## Step 5：逐一抓取岗位详情页（强制）

**执行前必读：`agents/detail-scraper.md`，按其中的流程执行。**

**核心原则：每个岗位的职责和要求必须来自详情页原文，不得从列表页推断。**

### 5a. 执行规范

1. 先将当前批次所有岗位的「名称 + URL」收集为完整清单
2. 逐一导航到每个详情页，提取「岗位职责」和「岗位要求」完整原文
3. 实时告知进度：「正在抓取 [i]/[total]：[岗位名称]」
4. **每抓完一个岗位立即追加写入临时文件**（见5b），防止数据丢失

### 5b. 数据持久化（防丢失）

每抓完一个岗位，立即执行：

```python
# 追加写入 /tmp/jobs_batch_N.json
import json, os
batch_file = f'/tmp/jobs_batch_{batch_num}.json'
existing = json.load(open(batch_file)) if os.path.exists(batch_file) else []
existing.append(job_data)
json.dump(existing, open(batch_file, 'w'), ensure_ascii=False, indent=2)
```

每批次完成后，汇总到 `/tmp/jobs_all.json`（去重后追加）。

### 5c. 详情页内容提取

```
1. 导航到详情页URL
2. get_page_text 读取全文
3. 识别职责段落：标题含「职责」「工作内容」「你将负责」「岗位描述」
4. 识别要求段落：标题含「要求」「资格」「你需要」「任职条件」
5. 提取完整内容，不裁剪，不改写
```

### 5d. 去重规则

写入临时文件前检查：若 URL 已存在于当前数据集，跳过，不重复写入。

### 5e. 降级处理

| 情况 | 处理 |
|------|------|
| 详情页打不开 | 跳过，记录「无法访问」，继续下一个 |
| 需要登录 | 提示用户在浏览器登录后点击继续，等待确认 |
| 有效岗位不足预期50% | 扩展关键词（使用Step 1的备选词）或放开城市限制 |

---

## Step 6：根据用户背景评级

**执行前必读：先读取 `prompts/scoring/README.md`，再读取 `prompts/scoring/example-mine.md`。**
理解示范文件的评级结构后，根据当前用户在 Step 1 提供的背景，动态生成本次专属的评级标准，再开始评级。不要直接套用示范文件的内容。

评级完全基于用户在 Step 1 提供的个人背景描述。

**评级逻辑：**
- 将用户背景与每个岗位的要求逐条对比
- 用户明确说"不想做X"，岗位主要工作是X → 直接标低
- 用户有明确优势且岗位恰好需要 → 标高
- 有技术硬门槛但用户背景不符 → 标低，不论其他内容多吸引人
- 岗位不一定与 AI 相关，评级标准完全跟随用户的实际目标方向

**评级结果：**
- 🟢 高（强烈推荐）：背景与岗位高度匹配，无明显短板
- 🟡 中（可以考虑）：基本匹配，有1-2项需补充或妥协
- 🔴 低（仅供参考）：核心要求与背景明显不符

每个岗位附一句匹配说明，例如：
> 「匹配点：岗位强调AI工具应用，与你熟练使用ChatGPT/Midjourney吻合；注意：要求基础SQL，需评估是否可接受」

### 细致模式：每批先输出文本预览

```
第[X]批（第[n1]-[n2]页）共[m]个岗位：

序号 | 岗位名称 | 城市 | 评级 | 匹配说明
1   | AI产品运营实习 | 上海 | 🟢高 | 匹配点：...
2   | 大模型数据标注 | 北京 | 🔴低 | 不符合原因：...
```

询问：「第[X]批预览完成，是否继续第[n2+1]-[n2+5]页？」用户确认后继续。

---

## Step 7：生成Excel输出

### 有bash_tool（Cowork / Claude Code）

将最终数据从 `/tmp/jobs_all.json` 传入脚本生成：

```bash
# 先找到脚本的实际路径
SKILL_DIR=$(find ~/.claude/skills -name "generate_excel.py" 2>/dev/null | head -1 | xargs dirname)
python3 "$SKILL_DIR/generate_excel.py" \
  --data /tmp/jobs_all.json \
  --output /mnt/user-data/outputs/[公司名]_岗位列表_[日期].xlsx
```

若 find 找不到，直接内联执行 Excel 生成代码（见脚本内容）。

### 无bash_tool（Web模式）

使用内联 openpyxl 代码生成。

### 表格结构

| 列 | 内容 |
|----|------|
| 序号 | 1, 2, 3... |
| 岗位名称 | 官网原文 |
| 岗位地点 | 城市 |
| 岗位职责 | 官网原文，完整不缩写 |
| 岗位要求 | 官网原文，完整不缩写 |
| 适合程度 | 高 / 中 / 低（带颜色） |
| 匹配说明 | 针对用户背景的说明 |
| 详情页链接 | 原始链接，方便直接申请 |

### 格式规范

- 第一行：公司名 + 搜索关键词 + 搜索日期 + 用户背景摘要（50字以内）
- 标题行：深蓝背景（1F3864）+ 白色加粗字体
- 适合程度颜色：高=绿（C6EFCE），中=黄（FFEB9C），低=红（FFC7CE）
- 列宽：按内容自适应（最小10，最大60）
- 文字自动换行，开启自动筛选，冻结标题行
- 最后统计行：🟢高X个 / 🟡中X个 / 🔴低X个 / 共X个
- 细致模式：所有批次合并为单一Excel，不输出多个文件

---

## 参考文件

- `references/company_urls.md`：主流公司招聘URL模板和参数说明，URL过期时参考此文件结构自行更新
- `parsers/`：各公司官网爬取逻辑，Step 2 前先查是否有对应文件（读 `parsers/README.md` 了解结构）
- `prompts/scoring/`：评级 prompt 示范，Step 6 执行前必读（读 `prompts/scoring/README.md`）
- `agents/detail-scraper.md`：Step 5 详情页抓取 agent 指令
- `templates/字节跳动AI岗位_推荐_.xlsx`：Excel样式模板，`generate_excel.py` 的样式以此为基准
- `scripts/generate_excel.py`：Excel生成脚本，含自适应列宽和去重逻辑
