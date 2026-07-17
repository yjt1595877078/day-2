# 步骤 22 — XML 数据导入功能

> **新增功能:** XML 数据导入 `/xml-import`

---

## 新增路由

| 路由 | 方法 | 说明 |
|------|:----:|------|
| `/xml-import` | GET | 显示 XML 导入页面 |
| `/xml-import` | POST | 接收 XML 数据，解析后返回 JSON |

### 功能说明

- 需登录才能访问
- 从表单接收 `xml_data` 参数
- 解析 XML 中的 `user` 节点，提取 `name` 和 `email`
- 以 JSON 格式返回解析结果

### 新增/修改文件

| 文件 | 说明 |
|------|------|
| `app.py` | 新增 `/xml-import` 路由 + `xml.etree.ElementTree`、`json` 导入 |
| `templates/xml_import.html` | **新增** XML 导入页面（文本框 + 导入按钮 + 结果展示） |
| `templates/base.html` | 导航栏新增"XML导入"链接 |
