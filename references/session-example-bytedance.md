# 实战案例：字节跳动 AI 实习岗位搜索（2026-04-14）

> **用途**：本文件是首次用此 skill 跑通完整流程的真实记录，包含可复用的 URL、执行规范、错误示范和岗位原文示例。
> 遇到新公司时，先读本文件，了解正确流程和常见坑。

---

## 一、用户背景与参数

```
公司：字节跳动
届别：2027届，日常实习（URL参数 type=3）
城市：上海 → 后扩展至北京、深圳
关键词：AI产品、AI运营
用户背景：文科背景，AI native，熟练使用AI工具（workflow/产品demo），
         偏应用方向，不做模型端，不会写代码
```

---

## 二、Step 2 URL 验活记录

### 有效 URL（已验证，2026-04-14）

```
# AI运营 · 上海 · 日常实习
https://jobs.bytedance.com/campus/position?keywords=AI%E8%BF%90%E8%90%A5&location=CT_10&type=3&current=1&limit=30

# AI产品 · 北京+上海+深圳 · 日常实习
https://jobs.bytedance.com/campus/position?keywords=AI%E4%BA%A7%E5%93%81&location=CT_10%2CCT_11%2CCT_100010&type=3&current=1&limit=30

# 仅北京 · 不限关键词
https://jobs.bytedance.com/campus/position?keywords=AI&location=CT_11&type=3&current=1&limit=30
```

### 城市代码速查

| 城市 | 代码 |
|------|------|
| 上海 | CT_10 |
| 北京 | CT_11 |
| 深圳 | CT_100010 |
| 杭州 | CT_9 |
| 多城市 | CT_10%2CCT_11（用%2C连接） |

### 网站访问注意

- `jobs.bytedance.com` **被 WebFetch 拦截**，必须用 Chrome MCP（Cowork 环境）
- 详情页通常**不需要登录**即可访问，列表页偶有限制
- 翻页推荐用 **URL 参数 `current=N`** 直接跳页，比点击"下一页"更稳定

---

## 三、Step 3 搜索框操作记录

字节跳动可以直接用 URL 参数搜索，**无需操作搜索框**，直接构造 URL 导航最高效（对应 SKILL.md Step 3 的方法C）。

若必须操作搜索框时（其他公司），推荐顺序：
1. `form_input`（传入元素ID + 关键词）→ 点击搜索按钮
2. `javascript_tool` 设值 + 触发 input 事件
3. 直接 URL 参数

---

## 四、Step 4 翻页记录

字节跳动翻页：修改 URL 中的 `current` 参数直接导航，**无需点击翻页按钮**。

验证翻页成功：对比翻页前后第一个岗位名称是否变化。

本次实际搜索量：
- "AI运营"：974 条结果，北京岗位占绝大多数
- "AI产品"：大量结果，北京同样为主

### ❌ 错误示范：只爬取当前页面

**本次首次执行犯了以下错误，后续必须避免：**

```
错误做法：
1. 导航到搜索结果页
2. 用 get_page_text / JavaScript 提取当前页可见的岗位（约30条）
3. 直接结束列表收集，开始抓详情页
4. 只抓到14个详情页，就认为完成了

问题：
- "AI运营"实际有 974 条结果、约32页
- 只看了第1页，漏掉了 ~960 条结果
- 输出的14个岗位远未覆盖用户真实需求
```

**正确做法（遵循 SKILL.md Step 4）：**

```
✅ 先读取总页数（从分页组件提取）
✅ 按 current=1, 2, 3... 逐页翻，每页收集岗位URL列表
✅ 所有页翻完后，再进入 Step 5 逐一抓取详情页
✅ 快速模式只翻前5页；细致模式每5页预览确认一次
✅ 翻页后验证成功（检查第一个岗位名称是否变化）
```

---

## 五、Step 5 数据持久化与失败处理

### 数据字段格式

抓取到的每条岗位写入 `/tmp/jobs_batch_N.json`，字段名如下（必须与 `generate_excel.py` 的 COLUMNS 定义一致）：

```json
{
  "title": "AI产品运营实习生-开发者服务",
  "location": "北京",
  "responsibilities": "1、参与Agent产品用户访谈……（官网原文，完整保留编号）",
  "requirements": "1、本科及以上学历在读……（官网原文，完整保留编号）",
  "level": "高",
  "match_note": "匹配点：完全运营和用户研究导向，Agent重度用户即可；无技术/编程要求",
  "url": "https://jobs.bytedance.com/campus/position/XXXXXX/detail"
}
```

| Excel列 | JSON字段 | 说明 |
|---------|---------|------|
| 岗位名称 | `title` | 官网原文 |
| 岗位地点 | `location` | 城市 |
| 岗位职责 | `responsibilities` | 详情页原文，含编号 |
| 岗位要求 | `requirements` | 详情页原文，含编号 |
| 适合程度 | `level` | "高"/"中"/"低" |
| 匹配说明 | `match_note` | 针对用户背景的一句话说明 |
| 详情页链接 | `url` | 原始链接 |

### 详情页抓取失败的处理规范

每个岗位详情页失败时，**不要直接跳过，先重试**：

```
第1次失败 → 等待2秒后重试同一URL
第2次失败 → 尝试不同方法（get_page_text → read_page → javascript_tool 提取）
第3次失败 → 记录为失败，继续下一个

重试期间告知用户进度：
「正在抓取 7/32：AI策略运营实习生（第2次尝试）」
```

所有岗位处理完后，若存在失败项，**在输出 Excel 之前告知用户**：

```
以下 3 个岗位详情页无法成功读取，已跳过：
1. AI数据分析实习生-商业产品 | https://jobs.bytedance.com/campus/position/7388xxx/detail
2. 增长运营实习生-抖音电商 | https://jobs.bytedance.com/campus/position/7401xxx/detail
3. AI内容策略实习生-飞书 | https://jobs.bytedance.com/campus/position/7412xxx/detail

你可以直接点击链接在浏览器中手动查看这些岗位，
将内容复制给我后我可以补充到Excel表格中。
```

---

## 六、Step 6 评级参考（本次用户背景）

> 用户背景：文科/非理工科，AI工具熟练（workflow/demo），不写代码，偏应用场景

### 🟢 高 — 典型案例（官网原文节选）

**AI生态和赛事运营-抖音**
- 职责：「参与2026年全国AI黑客松联赛整体运营；PMO职责；跨团队沟通」
- 要求：「本科及以上学历在读，**专业不限**；逻辑思维和沟通协调能力」
- 判断：明确专业不限，完全运营导向 ✅

**AI产品运营-开发者服务**
- 职责：「参与Agent产品用户访谈；用户故事内容包装与宣发；种子用户培养」
- 要求：「对AI Agent、AIGC有浓厚兴趣；文字功底扎实；**Agent深度使用用户优先**」
- 判断：重度AI用户即可，内容运营导向 ✅

**AI产品经理-Coze**
- 职责：「Agent效果评估；Prompt调优；App核心链路用户体验优化」
- 要求：「计算机科学、人工智能或相关专业优先（非必须）；**重度AI产品使用者优先**」
- 判断：Prompt调优属AI native能力，设计工具+数据分析即可 ✅

### 🟡 中 — 典型案例

**AI平台运营-中国交易与广告**
- 要求：「理解LLM及其应用场景，对AI效果度量、多Agent编排有实操经验者优先」
- 判断：需理解LLM概念但不需编程，AI应用场景拓展方向 ⚠️

### 🔴 低 — 典型案例（官网原文节选）

**AI数据运营-视频通话**
- 要求：「**必须**具备大模型产品、AI训练师、数据标注等相关实习经验；有过ASR、TTS等多模态模型相关项目经验；拥有Agent搭建和迭代经验，了解FC和RAG优先」
- 判断：硬性技术门槛，与用户背景明显不符 ❌

---

## 七、Step 7 Excel 生成命令

```bash
SKILL_DIR=$(find ~/.claude/skills -name "generate_excel.py" 2>/dev/null | head -1 | xargs dirname)
python3 "$SKILL_DIR/generate_excel.py" \
  --merge-batches \
  --company "字节跳动" \
  --keywords "AI产品,AI运营" \
  --output /mnt/user-data/outputs/字节跳动_AI实习岗位_2026-04-14.xlsx
```

---

## 八、本次实战统计

| 指标 | 数值 | 备注 |
|------|------|------|
| 搜索结果总量 | ~974条（AI运营） | 仅看了第1页，为错误示范 |
| 实际抓取岗位数 | 14 | 第1页中成功读取详情的数量 |
| 高适合度 | 5 个（36%）| |
| 中适合度 | 7 个（50%）| |
| 低适合度 | 2 个（14%）| |
| 北京岗位占比 | ~90% | AI运营结果以北京为主 |
| 上海岗位 | 少量 | 建议放开城市限制搜索 |
