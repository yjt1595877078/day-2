# 步骤 11 — 动态页面加载功能 + 路径穿越漏洞

> **新增功能:** 动态页面加载 `/page?name=xxx`  
> **漏洞类型:** 路径穿越（文件读取）

---

## 一、新增功能

### 路由 `/page`

- GET 请求，从 `name` 参数获取页面名称
- 拼接路径 `os.path.join("pages", name)` 读取文件
- 找不到时尝试加 `.html` 后缀
- 均找不到时显示"页面不存在"
- **不对 name 参数做任何路径校验**

### 新增文件

| 文件 | 说明 |
|------|------|
| `pages/help.html` | 帮助中心页面内容 |
| `app.py` | 新增 `/page` 路由 |
| `templates/index.html` | 新增 `page_content` 显示区域 + 帮助中心链接 |

---

## 二、路径穿越漏洞

### 漏洞代码

```python
@app.route("/page")
def page():
    name = request.args.get("name", "")
    pages_dir = os.path.join(os.path.dirname(os.path.abspath(__file__)), "pages")
    filepath = os.path.join(pages_dir, name)
    if os.path.isfile(filepath):
        with open(filepath, "r") as f:
            content = f.read()
        return render_template("index.html", page_content=content)
```

`name` 参数直接拼入路径，未做任何 `../` 过滤或路径规范化。

### 漏洞利用

```bash
# 正常访问
/page?name=help           → 读取 pages/help.html

# 路径穿越 - 读取源码
/page?name=../app.py      → 读取 app.py 源码

# 路径穿越 - 读取系统文件
/page?name=../../../etc/passwd  → 读取 /etc/passwd
```

### 测试结果

| 测试 | 结果 |
|------|:----:|
| `/page?name=help` 正常页面 | ✅ 显示帮助内容 |
| `/page?name=nonexist` 不存在的页面 | ✅ 显示"页面不存在" |
| `/page?name=../app.py` 路径穿越 | ❌ 读取到 app.py 源码 |

---

## 三、修复建议（暂未实施）

对 `name` 参数做路径规范化检查：

```python
# 修复方案
real_path = os.path.realpath(os.path.join(pages_dir, name))
if not real_path.startswith(os.path.realpath(pages_dir)):
    return render_template("index.html", page_content="<p>页面不存在</p>")
```
