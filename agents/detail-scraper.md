# Agent：详情页抓取器

**调用时机**：Step 5，获得岗位列表（名称 + URL）后，逐一抓取每个详情页。

---

## 任务目标

对列表中每个岗位执行：导航 → 读取 → 提取 → 写入，输出符合 `generate_excel.py` 字段规范的 JSON 数据。

---

## 执行流程

### 1. 初始化

接收参数：
- `jobs_list`：[{title, url}, ...] 当前批次岗位清单
- `batch_num`：批次编号（用于写入文件名）
- `total`：本批次总数

### 2. 逐一处理

对每个岗位按以下顺序执行：

**① 导航**
```
navigate 到详情页 URL
等待页面加载（至少1秒）
```

**② 读取页面内容**（按优先级尝试）
```
方法1：get_page_text 读取全文
方法2：read_page 获取可访问性树
方法3：javascript_tool 提取：
  document.querySelector('[class*="detail"],[class*="content"],[class*="job"]')?.innerText
```

**③ 提取字段**

| 字段 | 识别关键词 | 规则 |
|------|-----------|------|
| `responsibilities` | 「职责」「工作内容」「你将负责」「岗位描述」 | 完整原文，保留编号，不裁剪 |
| `requirements` | 「要求」「资格」「你需要」「任职条件」 | 完整原文，保留编号，不裁剪 |
| `location` | 城市字段 | 列表页已有则沿用，否则从详情页提取 |

**④ 写入临时文件**（每条立即写，防丢失）
```python
import json, os
batch_file = f'/tmp/jobs_batch_{batch_num}.json'
existing = json.load(open(batch_file)) if os.path.exists(batch_file) else []
existing.append(job_data)
json.dump(existing, open(batch_file, 'w'), ensure_ascii=False, indent=2)
```

**⑤ 汇报进度**
```
「正在抓取 {i}/{total}：{title}」
```

---

## 输出字段规范

每条记录必须包含以下字段，与 `generate_excel.py` 的 COLUMNS 定义严格对应：

```json
{
  "title": "岗位名称（官网原文）",
  "location": "城市",
  "responsibilities": "岗位职责完整原文（含编号）",
  "requirements": "岗位要求完整原文（含编号）",
  "level": "",
  "match_note": "",
  "url": "详情页原始链接"
}
```

> `level` 和 `match_note` 由 Step 6 填入，此处留空。

---

## 失败处理

```
第1次失败 → 等待2秒，重试同一URL
第2次失败 → 换方法（get_page_text → read_page → javascript_tool）
第3次失败 → 记录为失败项，title/url 保留，其余字段填「抓取失败」，继续下一个
```

进度汇报格式：
```
「正在抓取 7/32：AI策略运营实习生（第2次尝试）」
```

---

## 批次完成后

1. 将本批次文件汇总到 `/tmp/jobs_all.json`（去重后追加，按 URL 去重）
2. 若有失败项，汇总后告知用户：

```
以下 N 个岗位详情页无法读取，已跳过：
1. {title} | {url}
...
你可以点击链接手动查看，将内容复制给我后可补充到 Excel 中。
```

---

## 去重规则

写入前检查：若 `url` 已存在于当前 batch 文件，跳过，不重复写入。
