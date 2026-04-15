# 字节跳动爬取逻辑

最后验证：2026-04-14 | 环境：Claude Cowork（Chrome MCP）

---

## 访问限制

- `jobs.bytedance.com` **被 WebFetch 拦截**，必须用 Chrome MCP（Cowork 环境）
- 列表页偶有访问限制，详情页通常不需登录即可访问

---

## URL 模板

```
https://jobs.bytedance.com/campus/position?keywords=关键词&location=城市代码&type=届别&current=页码&limit=30
```

### 参数说明

| 参数 | 说明 | 示例 |
|------|------|------|
| `keywords` | 搜索关键词（URL编码） | `AI%E8%BF%90%E8%90%A5`（AI运营） |
| `location` | 城市代码，多城市用 `%2C` 连接 | `CT_10%2CCT_11` |
| `type` | 岗位类型 | `3`=日常实习 |
| `current` | 页码，从 1 开始 | `1` |
| `limit` | 每页数量，固定 30 | `30` |

### 城市代码

| 城市 | 代码 |
|------|------|
| 上海 | CT_10 |
| 北京 | CT_11 |
| 深圳 | CT_100010 |
| 杭州 | CT_9 |

### 常用示例

```
# AI运营 · 上海 · 日常实习
https://jobs.bytedance.com/campus/position?keywords=AI%E8%BF%90%E8%90%A5&location=CT_10&type=3&current=1&limit=30

# AI产品 · 北京+上海+深圳 · 日常实习
https://jobs.bytedance.com/campus/position?keywords=AI%E4%BA%A7%E5%93%81&location=CT_10%2CCT_11%2CCT_100010&type=3&current=1&limit=30
```

---

## 搜索方式

直接构造 URL 导航，**无需操作搜索框**（对应 SKILL.md Step 3 方法C）。
这是字节官网最稳定的方式，优先级高于 form_input 和 JS 注入。

---

## 翻页方式

修改 URL 中的 `current` 参数直接跳页，**无需点击翻页按钮**。

```
第1页：current=1
第2页：current=2
...
```

验证翻页成功：对比翻页前后第一个岗位名称是否变化。

---

## 数据规模参考

- "AI运营" 关键词：约 974 条结果（约 33 页），北京岗位占绝大多数
- "AI产品" 关键词：结果量级相近，北京同样为主
- 建议：若只搜上海结果偏少，可放开城市限制或扩展至北京、深圳
