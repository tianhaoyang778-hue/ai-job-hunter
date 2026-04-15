# 大厂校招网址与URL参数手册

## URL 模板说明

大多数大厂的搜索URL遵循以下规律：
- `keywords` = 搜索关键词（URL编码）
- `location` / `city` = 城市代码
- `type` / `job_type` = 实习类型（日常实习）
- `current` / `page` = 页码
- `limit` / `size` = 每页数量

---

## 字节跳动 (ByteDance)

**校招首页**：https://jobs.bytedance.com/campus

**搜索URL模板**：
```
https://jobs.bytedance.com/campus/position?keywords={关键词}&location={城市代码}&type=3&current=1&limit=30
```

**城市代码**：
- 上海：`CT_10`
- 北京：`CT_11`
- 深圳：`CT_100010`
- 杭州：`CT_9`
- 多城市：`CT_10%2CCT_11%2CCT_100010`（用%2C连接）
- 不限：留空

**实习类型**：`type=3`（日常实习）

**关键词编码**：
- AI产品：`AI%E4%BA%A7%E5%93%81`
- AI运营：`AI%E8%BF%90%E8%90%A5`
- AIGC运营：`AIGC%E8%BF%90%E8%90%A5`

**示例（AI运营·上海·日常实习）**：
```
https://jobs.bytedance.com/campus/position?keywords=AI%E8%BF%90%E8%90%A5&location=CT_10&type=3&current=1&limit=30
```

**岗位详情页格式**：
```
https://jobs.bytedance.com/campus/position/{岗位ID}/detail
```

---

## 腾讯 (Tencent)

**校招首页**：https://join.qq.com

**搜索URL模板**：
```
https://join.qq.com/post.html#!job_list?keyword={关键词}&category=&city={城市名}&job_type=2&job_tag=
```

**城市名**：直接用中文城市名，如 `上海`、`北京`、`深圳`

**实习类型**：`job_type=2`（实习）

**示例（AI产品·上海）**：
```
https://join.qq.com/post.html#!job_list?keyword=AI产品&city=上海&job_type=2
```

**备用API（更稳定）**：
```
https://join.qq.com/api/recruitment/campus/post?keyword=AI产品&city=上海&type=2&pageSize=30&pageNo=1
```

---

## 阿里巴巴 (Alibaba)

**校招首页**：https://talent.alibaba.com/campus

**搜索URL模板**：
```
https://talent.alibaba.com/campus?keywords={关键词}&city={城市名}&workTypes=1
```

**实习类型**：`workTypes=1`

**示例**：
```
https://talent.alibaba.com/campus?keywords=AI运营&city=上海&workTypes=1
```

---

## 百度 (Baidu)

**校招首页**：https://talent.baidu.com/external/baidu/campus.html

**搜索URL模板**：
```
https://talent.baidu.com/external/baidu/campus.html#/position-list?position_type=3&keywords={关键词}&city={城市名}
```

**实习类型**：`position_type=3`（实习）

**示例**：
```
https://talent.baidu.com/external/baidu/campus.html#/position-list?position_type=3&keywords=AI产品&city=上海
```

---

## 美团 (Meituan)

**校招首页**：https://campus.meituan.com

**搜索URL模板**：
```
https://campus.meituan.com/jobs?keyword={关键词}&city={城市名}&pageNo=1
```

---

## 快手 (Kuaishou)

**校招首页**：https://campus.kuaishou.cn

**搜索URL模板**：
```
https://campus.kuaishou.cn/#/campus-recruitment/job-list?jobName={关键词}&cityName={城市名}&jobType=INTERNSHIP
```

---

## 网易 (NetEase)

**校招首页**：https://campus.163.com

**搜索页**：
```
https://campus.163.com/school/position/index.html?keyword={关键词}&city={城市名}
```

---

## 京东 (JD.com)

**校招首页**：https://campus.jd.com

**搜索**：需要在首页手动选择城市+关键词，URL参数不稳定，建议直接访问首页后用JavaScript提取

---

## 小米 (Xiaomi)

**校招首页**：https://hr.xiaomi.com/campus

**搜索**：
```
https://hr.xiaomi.com/campus/jobs?keyword={关键词}&city={城市名}&positionType=internship
```

---

## 通用备用策略

如果上述URL不工作（网站改版）：
1. 用 `WebSearch` 搜索「[公司名] 校园招聘官网 实习 AI 2025」
2. 进入官网主页，用Chrome MCP截图查看搜索框，然后手动搜索
3. 或搜索「[公司名] 内推 AI实习 2027届 牛客网」在牛客找汇总帖
