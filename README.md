# SmartDNS GitHub 域名加速规则

自动从 [Loyalsoldier/v2ray-rules-dat](https://github.com/Loyalsoldier/v2ray-rules-dat) 的 `proxy-list.txt` 过滤出 GitHub 相关域名，并从 [GitHub520](https://github.com/521xueweihan/GitHub520) 拉取静态 hosts，转换为 SmartDNS 可直接使用的格式。

**更新频率**：每天凌晨 2:30（GitHub Actions cron: `30 2 * * *`）

---

## 📦 仓库文件

| 文件 | 格式 | 说明 |
|------|------|------|
| `github-domains.list` | 每行一个纯域名 | 用于 SmartDNS 下载设置 → 域名规则列表 |
| `github-hosts.smartdns.conf` | `nameserver /域名/github` | SmartDNS `conf-file` 直接 include，走 github 服务器组 |
| `github-hosts-address.smartdns.conf` | `address /域名/IP` | GitHub520 静态 IP 兜底，可与 nameserver 同时使用 |

---

## 🔗 jsDelivr 加速 URL（国内拉最快）

```
github-domains.list          https://fastly.jsdelivr.net/gh/<你的GitHub用户名>/SmartDNS-GitHub-Domains@master/github-domains.list
github-hosts.smartdns.conf   https://fastly.jsdelivr.net/gh/<你的GitHub用户名>/SmartDNS-GitHub-Domains@master/github-hosts.smartdns.conf
github-hosts-address.smartdns.conf https://fastly.jsdelivr.net/gh/<你的GitHub用户名>/SmartDNS-GitHub-Domains@master/github-hosts-address.smartdns.conf
```

> 把 `<你的GitHub用户名>` 替换成你实际的（比如 `InvSuper`）。fastly 是 jsDelivr 的 Fastly 加速节点，国内拉秒级。

---

## 🛠 OpenWrt SmartDNS 配置

### 方式 1：下载设置 + 域名规则列表（推荐，LuCI 原生支持）

**服务 → SmartDNS → 域名规则设置 → 域名规则列表 → 添加**：
```
域名规则名称：github-rule
服务器组：github
域名列表文件：填上面的 jsDelivr URL（github-domains.list）
```

然后 **下载设置 tab** 添加定时拉取：
```
URL：https://fastly.jsdelivr.net/gh/<你的GitHub用户名>/SmartDNS-GitHub-Domains@master/github-domains.list
定时间隔：1440 分钟（每天一次）
格式：纯域名列表
```

### 方式 2：conf-file 直接 include（更简单）

**服务 → SmartDNS → 自定义配置** 里加一行：
```
conf-file https://fastly.jsdelivr.net/gh/<你的GitHub用户名>/SmartDNS-GitHub-Domains@master/github-hosts.smartdns.conf
```

或者先下载到本地再 include：
```sh
# OpenWrt 定时拉 + 重载
curl -fsSL "https://fastly.jsdelivr.net/gh/<你的GitHub用户名>/SmartDNS-GitHub-Domains@master/github-hosts.smartdns.conf" \
  -o /etc/smartdns/github-hosts.smartdns.conf
/etc/init.d/smartdns reload
```

---

## ⚠️ 前提条件

**SmartDNS 里必须先建好 github 服务器组**（走海外 DNS，比如 Cloudflare/Google DoH）：

```
github 组服务器：
  https://1.1.1.1/dns-query     (Cloudflare)
  https://dns.google/dns-query  (Google)
  https://dns.adguard.com/dns-query  (AdGuard)

github 组测速：
  测速模式：TCP
  测速端口：443
  超时阈值：200ms
  测速频率：每小时
```

**重要**：把 GitHub 域名从 global-domains.list 里删掉，否则永远走国内 DNS（污染 IP）。

---

## 🧪 验证

```sh
# OpenWrt 终端
# 1. 看 github 域名列表
cat github-domains.list | head

# 2. 看 SmartDNS 日志里 github.com 是不是走 github 组
logread -f | grep github
# 客户端访问 github.com 的同时看日志

# 3. 实测下载速度
time wget -O /dev/null https://raw.githubusercontent.com/521xueweihan/GitHub520/main/hosts
```

---

## 🔄 手动触发更新

GitHub Actions → Actions tab → `GitHub Domain List → SmartDNS` → **Run workflow**

---

## 📜 License

MIT
