# SmartDNS 域名加速规则（GitHub + Global 双份）

自动从 [Loyalsoldier/v2ray-rules-dat](https://github.com/Loyalsoldier/v2ray-rules-dat) 定时生成，SmartDNS 可直接使用。

**更新频率**：每天凌晨 2:30（GitHub Actions cron: `30 2 * * *`）

---

## 📦 仓库文件

### github 组专用（只有 GitHub 域名）

| 文件 | 格式 | 说明 |
|------|------|------|
| `github-domains.list` | 每行一个纯域名 | SmartDNS 下载设置 → **域名规则列表** 用 |
| `github-hosts.smartdns.conf` | `nameserver /域名/github` | SmartDNS `conf-file` 直接 include |
| `github-hosts-address.smartdns.conf` | `address /域名/IP` | GitHub520 静态 IP 兜底，可与 nameserver 同时使用 |

### global 组专用（国内加速，已剔除 GitHub）

| 文件 | 格式 | 说明 |
|------|------|------|
| `global-domains.list` | 每行一个纯域名 | SmartDNS 下载设置 → **域名规则列表** 用 |
| `global-hosts.smartdns.conf` | `nameserver /域名/global` | SmartDNS `conf-file` 直接 include |

> **重要**：global-domains.list 已**自动剔除所有 GitHub 相关域名**，因为 GitHub 域名应该走 github 组（海外 DNS），否则会被 GFW 污染。

---

## 🔗 jsDelivr 加速 URL（国内拉最快）

把 `<你的GitHub用户名>` 替换成你实际的（比如 `InvSuper`）。fastly 是 jsDelivr 的边缘加速，国内拉秒级。

```
github 组：
  github-domains.list               https://fastly.jsdelivr.net/gh/<用户名>/SmartDNS-GitHub-Domains@master/github-domains.list
  github-hosts.smartdns.conf       https://fastly.jsdelivr.net/gh/<用户名>/SmartDNS-GitHub-Domains@master/github-hosts.smartdns.conf
  github-hosts-address.smartdns.conf https://fastly.jsdelivr.net/gh/<用户名>/SmartDNS-GitHub-Domains@master/github-hosts-address.smartdns.conf

global 组：
  global-domains.list               https://fastly.jsdelivr.net/gh/<用户名>/SmartDNS-GitHub-Domains@master/global-domains.list
  global-hosts.smartdns.conf        https://fastly.jsdelivr.net/gh/<用户名>/SmartDNS-GitHub-Domains@master/global-hosts.smartdns.conf
```

> ⚠️ **jsDelivr 有 7 天缓存**，想强制刷新可以在 URL 后加 `?v=时间戳` 参数，比如 `https://fastly.jsdelivr.net/gh/.../github-domains.list?v=20261106`。

---


