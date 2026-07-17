# 步骤 24 — XXE 漏洞修复

> **修复日期:** 2026-07-14  
> **修复方式:** 移除 XXE 实体处理逻辑

---

## 修复详情

### 修复内容

删除 XML 导入功能中主动解析 `<!ENTITY ... SYSTEM>` 并读取本地文件的代码块。

```python
# 移除前（存在 XXE 漏洞）
xxe_pattern = re_mod.findall(r'<!ENTITY\s+(\w+)\s+SYSTEM\s+"([^"]+)"', xml_data)
for entity_name, filepath in xxe_pattern:
    if filepath.startswith("file://"):
        filepath = filepath[7:]
    with open(filepath, "r") as f:
        content = f.read()
    xml_data = xml_data.replace(f"&{entity_name};", content)

# 移除后（安全）
# 直接解析 XML，不处理任何实体定义
root = ET.fromstring(xml_data)
```

---

## 修复验证

| 测试项 | 修复前 | 修复后 |
|--------|:------:|:------:|
| 正常 XML 解析 | ✅ | ✅ 正常 |
| XXE 读取 `/etc/passwd` | ❌ 读取成功 | ✅ 已拦截 |
| XXE 读取 `app.py` | ❌ 读取成功 | ✅ 已拦截 |
