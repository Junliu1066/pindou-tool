# 拼豆图纸工具 · Windows 服务器部署教程（零基础版）

> 目标：让任何人打开 https://junliu.online 就能用你的工具。
> 方案：Caddy（一个小程序，自动配 HTTPS）。全程约 20 分钟。

---

## 准备：你需要的东西

- ✅ Windows 服务器（你已有）
- ✅ 域名 junliu.online（你已有）
- ✅ 服务器的「公网 IP」（在你买服务器的后台能看到，例如阿里云/腾讯云控制台）
- ✅ 网站文件：`index.html`、`admin.html`

---

## 第 0 步 · 部署前改密码（重要）

用记事本打开 `admin.html`，找到这一行：

```
const ADMIN_PW='admin888';
```

把 `admin888` 改成你自己的密码，保存。否则别人也能进你的后台。

---

## 第 1 步 · 域名解析（让 junliu.online 指向你的服务器）

去你买域名的网站（阿里云/腾讯云/Namesilo 等）→ 找到「域名解析 / DNS」→ 添加两条记录：

| 类型 | 主机记录 | 记录值 |
|------|---------|--------|
| A    | @       | 你服务器的公网 IP |
| A    | www     | 你服务器的公网 IP |

保存。解析生效一般几分钟到半小时。
验证：在自己电脑上按 `Win+R` → 输入 `cmd` → 回车 → 打 `ping junliu.online`，
如果显示的 IP 是你服务器的 IP，就成了。

---

## 第 2 步 · 开放端口（让外网能访问）

HTTPS 网站要用 80 和 443 两个端口，要开两个地方：

**A. 云服务器后台「安全组 / 防火墙」**
在你买服务器的控制台找到「安全组」，添加入站规则：放行 **TCP 80** 和 **TCP 443**。

**B. Windows 自带防火墙**
服务器上「开始」搜「高级安全 Windows Defender 防火墙」→ 入站规则 → 新建规则
→ 端口 → TCP → 特定端口填 `80,443` → 允许连接 → 完成。

---

## 第 3 步 · 放网站文件

1. 在服务器 C 盘新建文件夹 `C:\pindou`
2. 把 `index.html` 和 `admin.html` 复制进去
   （怎么传上去？远程桌面连服务器后，直接从你电脑复制粘贴进去即可。）

---

## 第 4 步 · 装 Caddy 并启动

1. 浏览器打开 https://caddyserver.com/download
   选 Platform = **windows / amd64** → 点 **Download** → 得到一个 `caddy.exe`
2. 把 `caddy.exe` 也放进 `C:\pindou`
3. 把项目里的 `Caddyfile` 文件也放进 `C:\pindou`（注意：文件名就叫 `Caddyfile`，没有后缀）
4. 在 `C:\pindou` 文件夹空白处按住 `Shift` + 右键 → 「在此处打开 PowerShell / 终端」
5. 输入并回车：

   ```powershell
   .\caddy.exe run
   ```

6. 第一次启动它会自动去申请 HTTPS 证书，看到类似 `serving initial configuration` 就成功了。
   **保持这个窗口开着**，关了网站就停了。

现在用手机或电脑浏览器打开 **https://junliu.online**，应该能看到工具了 🎉

---

## 第 5 步（可选但推荐）· 让它开机自启、后台常驻

上面那种「窗口开着才运行」不适合长期。让 Caddy 变成 Windows 服务：

在 `C:\pindou` 的 PowerShell（用**管理员身份**打开）里执行：

```powershell
# 安装为系统服务（需要联网，一次即可）
.\caddy.exe add-package github.com/caddyserver/caddy-service  # 若提示不支持可跳过，用下面 nssm 方案

# 推荐用 nssm（更简单）：
# 1) 下载 nssm：https://nssm.cc/download，解压拿到 nssm.exe 放进 C:\pindou
# 2) 管理员 PowerShell 执行：
.\nssm.exe install Caddy "C:\pindou\caddy.exe" "run --config C:\pindou\Caddyfile"
.\nssm.exe start Caddy
```

这样服务器重启后网站会自动起来，不用手动开窗口。

---

## 看访问量（你的「使用量统计」）

Caddy 默认不写访问日志。想统计访问量，在 `Caddyfile` 里加一段：

```
junliu.online, www.junliu.online {
    root * C:\pindou
    file_server
    encode gzip
    log {
        output file C:\pindou\access.log
    }
}
```

改完重启 Caddy。`access.log` 里每一行就是一次访问，越多说明用的人越多。
后台 `admin.html` 里有进一步分析的说明。

---

## 更新网站（改了 bug 之后）

1. 本地改好 `index.html`，`git commit`（见 README）
2. 远程桌面连服务器，把新的 `index.html` 覆盖 `C:\pindou` 里的旧文件
3. 用户刷新页面就是新版（静态文件，不用重启 Caddy）

---

## 卡住了怎么办

- 打不开网站：先确认第 1 步 `ping` 通、第 2 步两个端口都开了。
- 浏览器报「不安全」：等几分钟，证书还在申请；或确认 443 端口开了。
- 还是不行：把 PowerShell 窗口里红色的报错整段复制下来，发给我，我帮你看。
