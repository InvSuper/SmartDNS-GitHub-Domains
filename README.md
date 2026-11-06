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

## 🛠 OpenWrt SmartDNS 配置

### 前提：先建好两个服务器组

**github 组**（走海外 DNS，必须返回 GitHub 真实 IP）：
```
上游 DNS（选 2-3 个）：
  https://dns.adguard.com:3000/dns-query      (AdGuard DoH 非标准端口，防污染)
  https://1.1.1.1:8080/dns-query               (Cloudflare DoH 非标准端口)
  dns.adguard.com:853                           (AdGuard DoT 853)

测速模式：TCP 443
超时阈值：200ms
```

**global 组**（走国内 DNS）：
```
上游 DNS（选 3-4 个）：
  223.5.5.5                                     (阿里 DNS)
  119.29.29.29                                  (腾讯 DNS)
  180.184.1.1                                   (字节 DNS)
  1.1.1.1                                       (Cloudflare 备用)

测速模式：ping,tcp:443,tcp:80
超时阈值：300ms
```

### 方式 1：下载设置 + 域名规则列表（推荐，LuCI 原生）

**服务 → SmartDNS → 域名规则设置 → 域名规则列表 → 添加**，分别加两条：

```
# 规则 1：github 域名规则（优先级高）
域名规则名称：github-rule
服务器组：github
域名列表文件：填 github-domains.list 的 jsDelivr URL

# 规则 2：global 域名规则（兜底）
域名规则名称：global-rule
服务器组：global
域名列表文件：填 global-domains.list 的 jsDelivr URL
```

然后 **下载设置 tab** 添加定时拉取：
```
名称：GitHub 域名列表
URL：https://fastly.jsdelivr.net/gh/<用户名>/SmartDNS-GitHub-Domains@master/github-domains.list
定时间隔：720 分钟（每 12 小时）

名称：Global 域名列表
URL：https://fastly.jsdelivr.net/gh/<用户名>/SmartDNS-GitHub-Domains@master/global-domains.list
定时间隔：720 分钟
```

### 方式 2：conf-file 直接 include（简单粗暴）

**服务 → SmartDNS → 自定义配置** 里加两行（先下载到本地）：

```sh
# OpenWrt 终端，定时拉 + 重载
curl -fsSL "https://fastly.jsdelivr.net/gh/<用户名>/SmartDNS-GitHub-Domains@master/github-hosts.smartdns.conf" \
  -o /etc/smartdns/github-hosts.smartdns.conf
curl -fsSL "https://fastly.jsdelivr.net/gh/<用户名>/SmartDNS-GitHub-Domains@master/global-hosts.smartdns.conf" \
  -o /etc/smartdns/global-hosts.smartdns.conf
/etc/init.d/smartdns reload
```

然后自定义配置里 include：
```
conf-file /etc/smartdns/github-hosts.smartdns.conf
conf-file /etc/smartdns/global-hosts.smartdns.conf
```

### 方式 3：域名地址 + nameserver 双保险（推荐）

**服务 → SmartDNS → 域名地址** tab 里直接添加 GitHub520 静态 IP 兜底：
```sh
# 也可以直接 conf-file
conf-file /etc/smartdns/github-hosts-address.smartdns.conf
```

这样即使海外 DNS 全挂，静态 IP 兜底也能保证 GitHub 能访问。

---

## ⚠️ 关键注意事项

### 1. 不要把 GitHub 域名留在 global-domains.list 里！
GitHub 域名如果走国内 DNS（global 组），GFW 会返回污染 IP（123.45.67.89 这种假 IP），SmartDNS 永远测不到真实 IP。这个仓库的 `global-domains.list` 已经自动剔除了 github 相关域名，但**你原来本地的** `/etc/smartdns/domain-set/global-domains.list` 也要手动清理：

```sh
cp /etc/smartdns/domain-set/global-domains.list /etc/smartdns/domain-set/global-domains.list.bak
sed -i '/github\.ai\|github\.blog\|github\.career\|github\.com\|github\.community\|github\.dev\|github\.io\|github1s\.com\|githubapp\.com\|githubassets\.com\|githubcopilot\.com\|githubhackathon\.com\|githubhelp\.com\|githubnext\.com\|githubpreview\.dev\|githubstatus\.com\|githubsupport\.com\|githubuniverse\.com\|githubusercontent\.com\|rawgithub\.com\|thegithubshop\.com/d' /etc/smartdns/domain-set/global-domains.list
/etc/init.d/smartdns restart

# 验证
grep -i github /etc/smartdns/domain-set/global-domains.list && echo "还有残留！" || echo "清理干净 ✓"
```

### 2. github 组 DNS 必须返回真实 IP
国内 DoH（阿里云/DNSPod）对 GitHub 这种海外域名没用——GFW 在国内根服务器层面就替换污染了。**github 组必须用海外 DoH/DNS**：
- `dns.adguard.com:3000` — 非标准端口 DoH，防 SNI 阻断
- `1.1.1.1:8080` — Cloudflare 非标准端口 DoH
- `dns.adguard.com:853` — DoT，GFW 对 TLS 阻断弱

### 3. 测速模式必须 TCP 443
不要用 ICMP ping 测 GitHub——GitHub 防火墙可能拦 ping。用 `tcp:443,tcp:80,ping` 多重测速，SmartDNS 会自动选最优。

---

## 🧪 验证

```sh
# OpenWrt 终端
# 1. 看 github 域名列表
cat github-domains.list | head

# 2. 看 SmartDNS 日志里 github.com 是不是走 github 组
logread -f | grep github
# 客户端访问 github.com 的同时看日志，应该出现 "group: github"

# 3. 看 global 域名是不是走 global 组
logread -f | grep -E "baidu.com|qq.com"
# 应该出现 "group: global"

# 4. 实测下载速度
time wget -O /dev/null https://raw.githubusercontent.com/521xueweihan/GitHub520/main/hosts
# 预期速度 >2MB/s（国内正常 5-20MB/s）

# 5. 看服务器成功率（SmartDNS 管理页 → 服务器状态）
# github 组：成功率应该 >90%（不会是 0%）
# global 组：成功率应该 100%
```

---

## 🔄 手动触发更新

GitHub Actions → Actions tab → `GitHub Domain List → SmartDNS` → **Run workflow**

---

## 📋 完整生成逻辑

```
proxy-list.txt (Loyalsoldier)  →  grep github  →  github-domains.list  →  github-hosts.smartdns.conf
                                                           ↘ 合并
github-domains-set.txt  ←────  grep github ────────────────┘      ↓
                                                                  china-list.txt (Loyalsoldier)
                                                                  ↓ grep -ivFf
                                                           global-domains.list  →  global-hosts.smartdns.conf
                                                                  ↓
gh520.hosts (GitHub520)  →  awk IP→domain  →  github-hosts-address.smartdns.conf
```

---

## 📜 License

MIT
