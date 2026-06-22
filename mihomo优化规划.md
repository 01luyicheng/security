# Mihomo 配置优化实施计划（v2.0 最终版）

## 一、当前配置状态

| 项目 | 当前值 | 状态 |
|------|--------|------|
| TUN Stack | system | 稳定，推荐 |
| TUN MTU | 1400 | 已修正，匹配路径MTU |
| IPv6 DNS | true | 经分析应关闭，当前网络未就绪 |
| TUN IPv6 地址 | 配置但未生效 | 应注释，待网络就绪后开启 |
| tcp-concurrent | 未开启 | 应开启，为未来域名节点做准备 |
| store-fake-ip | 已开启 | 正常 |
| fake-ip-range | 198.18.0.1/16 | mihomo默认fake-ip网段，与inet4-address分离 |
| 规则覆盖 | 基础+补充 | 需绑定策略组 |

---

## 二、优化目标

1. **修复现有问题**：关闭未就绪的IPv6、修复fake-ip网段重叠
2. **为未来扩展做准备**：开启tcp-concurrent、预建策略组
3. **性能与体验**：优化MTU、规则策略组化
4. **安全加固**：已完成（secret、allow-lan、文件权限）

---

## 三、具体修改项

### 3.1 TUN MTU

**当前值**：`mtu: 1400`（已修正，匹配实测路径MTU=1400）

**结论**：保持 `1400`。移动笔记本场景下，不应超过可能遇到的最小路径MTU。

```yaml
tun:
  enable: true
  stack: system
  gso: true
  mtu: 1400
  inet4-address: 198.18.0.1/30
  # inet6-address:  # 待网络和节点均支持IPv6后再开启
  #   - fdfe:dcba:9876::1/126
  strict-route: true
  dns-hijack:
    - any:53
  auto-route: true
  auto-detect-interface: true
```

---

### 3.2 IPv6 策略：当前应关闭

**关键发现**：
- 本机无全局IPv6路由，仅有link-local地址
- 代理节点仅有IPv4（20.214.224.156）
- 开启`ipv6: true`后，双栈应用优先尝试IPv6 → 超时fallback到IPv4，**每次连接增加150~300ms延迟**
- 日志中反复出现`network is unreachable`

**结论**：当前弊大于利，应关闭。待以下条件满足后再开启：
1. 物理网络具备稳定的全局IPv6接入
2. 代理节点支持IPv6
3. `inet6-address`在Meta接口上正确生效

```yaml
dns:
  enable: true
  ipv6: false          # 当前网络未就绪，关闭
  enhanced-mode: fake-ip
  fake-ip-range: 198.19.0.1/16   # 修正：避免与inet4-address重叠
  listen: 127.0.0.1:53
```

---

### 3.3 启用 tcp-concurrent

**分析**：
- 当前单IPv4裸IP节点：tcp-concurrent对代理出站无收益（裸IP不触发多记录）
- **未来添加域名节点后**：域名通常解析到多A记录（CDN/Anycast），tcp-concurrent并发取最快握手成功的IP，对代理连接建立是正面影响
- 若域名节点支持双栈，tcp-concurrent相当于内置Happy Eyeballs，自动择优

**结论**：现在开启，为未来域名节点扩展做准备。当前日志中的`network is unreachable`来自直连IPv6失败，关闭IPv6 DNS后该问题会消失。

```yaml
tcp-concurrent: true
```

---

### 3.4 store-fake-ip

**状态**：已开启，维持现状。

- fake-ip映射的是域名→虚拟IP，不绑定具体节点
- url-test自动切换节点后，已有fake-ip仍然有效
- 65,531个IP对日常使用足够

---

### 3.5 常见网站规则（策略组化）

**核心变更**：将AI服务和流媒体规则从硬编码`PROXY`改为绑定策略组，降低未来修改成本。

#### 高优先级规则

| 规则 | 策略 | 理由 |
|------|------|------|
| `DOMAIN-SUFFIX,steamcontent.com,DIRECT` | DIRECT | Steam下载国内CDN |
| `DOMAIN-SUFFIX,steampipe.akamaized.net,DIRECT` | DIRECT | Steam下载CDN |
| `DOMAIN-SUFFIX,epicgamescdn.com,DIRECT` | DIRECT | Epic下载CDN |
| `DOMAIN-SUFFIX,download.epicgames.com,DIRECT` | DIRECT | Epic下载CDN |
| `DOMAIN-SUFFIX,epicgames-download1.akamaized.net,DIRECT` | DIRECT | Epic下载CDN |
| `DOMAIN-SUFFIX,iosapps.itunes.apple.com,DIRECT` | DIRECT | Apple更新CDN |
| `DOMAIN-SUFFIX,osxapps.itunes.apple.com,DIRECT` | DIRECT | Apple更新CDN |
| `DOMAIN-SUFFIX,appldnld.apple.com,DIRECT` | DIRECT | Apple更新CDN |
| `DOMAIN-SUFFIX,swcdn.apple.com,DIRECT` | DIRECT | Apple更新CDN |
| `DOMAIN-SUFFIX,updates.cdn-apple.com,DIRECT` | DIRECT | Apple更新CDN |
| `DOMAIN-SUFFIX,discord.com,PROXY` | PROXY | Discord被墙 |
| `DOMAIN-SUFFIX,discordapp.com,PROXY` | PROXY | Discord子域名 |
| `DOMAIN-SUFFIX,discord.gg,PROXY` | PROXY | Discord邀请 |
| `DOMAIN-SUFFIX,discord.media,PROXY` | PROXY | Discord媒体 |
| `DOMAIN-SUFFIX,discordcdn.com,PROXY` | PROXY | Discord CDN |
| `DOMAIN-SUFFIX,login.live.com,PROXY` | PROXY | 微软服务登录 |
| `DOMAIN-SUFFIX,login.microsoftonline.com,PROXY` | PROXY | 微软服务登录 |
| `DOMAIN-SUFFIX,account.microsoft.com,PROXY` | PROXY | 微软账户 |
| `DOMAIN-SUFFIX,logincdn.msauth.net,PROXY` | PROXY | 微软登录CDN |
| `DOMAIN-SUFFIX,openai.com,AI` | AI | ChatGPT/OpenAI |
| `DOMAIN-SUFFIX,chatgpt.com,AI` | AI | ChatGPT |
| `DOMAIN-SUFFIX,anthropic.com,AI` | AI | Claude |
| `DOMAIN-SUFFIX,claude.ai,AI` | AI | Claude主站 |
| `DOMAIN-SUFFIX,gemini.google.com,AI` | AI | Google AI |
| `DOMAIN-SUFFIX,perplexity.ai,AI` | AI | Perplexity |
| `DOMAIN-SUFFIX,copilot.microsoft.com,AI` | AI | Copilot |
| `GEOSITE,reddit,PROXY` | PROXY | Reddit |
| `DOMAIN-SUFFIX,onedrive.live.com,PROXY` | PROXY | OneDrive国际版 |
| `DOMAIN-SUFFIX,wikipedia.org,PROXY` | PROXY | Wikipedia |
| `DOMAIN-SUFFIX,wikimedia.org,PROXY` | PROXY | Wikimedia |

#### 中优先级规则（流媒体绑定STREAM组）

| 规则 | 策略 | 理由 |
|------|------|------|
| `GEOSITE,netflix,STREAM` | STREAM | Netflix |
| `GEOSITE,youtube,STREAM` | STREAM | YouTube |
| `GEOSITE,spotify,STREAM` | STREAM | Spotify |
| `GEOSITE,disney,STREAM` | STREAM | Disney+ |
| `DOMAIN-SUFFIX,max.com,STREAM` | STREAM | HBO Max |

#### 不推荐全量代理的服务

- `GEOSITE,apple`：包含`mzstatic.com`（App Store下载国内CDN），全量代理会降低更新速度
- `GEOSITE,steam`：Steam社区/商店需代理，但下载应直连。已用`steamcontent.com`等规则覆盖

---

### 3.6 预建策略组

现在预建策略组，未来添加节点时只需在`proxies`和`AUTO`组中追加，无需改动规则。

```yaml
proxy-groups:
  - name: "PROXY"
    type: select
    proxies:
      - "AUTO"
      - "Azure-VLESS-Reality"
      - "DIRECT"

  - name: "AUTO"
    type: url-test
    url: "https://www.google.com/generate_204"
    interval: 300
    tolerance: 50
    proxies:
      - "Azure-VLESS-Reality"
      # 未来域名节点自动在此扩展：
      # - "hk-node"
      # - "jp-node"
      # - "us-node"

  - name: "AI"
    type: select
    proxies:
      - "Azure-VLESS-Reality"
      - "AUTO"
      - "DIRECT"

  - name: "STREAM"
    type: select
    proxies:
      - "Azure-VLESS-Reality"
      - "AUTO"
      - "DIRECT"

  - name: "Fallback"
    type: fallback
    proxies:
      - "Azure-VLESS-Reality"
      - "DIRECT"
    url: "https://www.google.com/generate_204"
    interval: 30
```

---

## 四、实施步骤

### Step 1：备份当前配置

```bash
sudo cp /etc/mihomo/config.yaml /etc/mihomo/config.yaml.bak.$(date +%Y%m%d)
```

### Step 2：应用配置修改

1. 在配置顶部添加 `tcp-concurrent: true`
2. 修改 `dns.ipv6` 为 `false`
3. 修改 `dns.fake-ip-range` 为 `198.19.0.1/16`
4. 注释 `tun.inet6-address`
5. 修改规则策略：AI服务改为`AI`，流媒体改为`STREAM`
6. 预建 `AUTO`/`AI`/`STREAM` 策略组

### Step 3：验证与重启

```bash
# 验证配置语法（注意加 -d 参数）
sudo mihomo -t -f /etc/mihomo/config.yaml -d /etc/mihomo

# 重启服务（TUN参数修改后建议重启而非热重载）
sudo systemctl restart mihomo
```

### Step 4：验证项目

| 验证项 | 方法 | 预期结果 |
|--------|------|----------|
| MTU | `ip addr show Meta \| grep mtu` | `mtu 1400` |
| IPv6关闭 | `dig @127.0.0.1 google.com AAAA` | 无AAAA记录返回 |
| fake-ip范围 | 检查配置 | `198.19.0.1/16` |
| tcp-concurrent | 检查配置 | 存在 |
| 规则生效 | 访问`chatgpt.com` | 日志显示走`AI`策略组 |
| 策略组 | 访问`netflix.com` | 日志显示走`STREAM`策略组 |

---

## 五、风险与回滚

| 风险 | 概率 | 缓解措施 |
|------|------|----------|
| IPv6关闭后部分IPv6-only应用无法访问 | 低 | 当前网络本无IPv6，无实际影响；未来网络支持后重新开启 |
| fake-ip-range变更后fake-ip映射重置 | 低 | store-fake-ip映射会重新分配，对使用无影响 |
| 策略组切换导致某节点异常 | 低 | 策略组默认包含`DIRECT`回退选项 |
| 规则修改后某网站访问异常 | 低 | 检查规则优先级，必要时调整 |

**回滚方案**：

```bash
sudo cp /etc/mihomo/config.yaml.bak.YYYYMMDD /etc/mihomo/config.yaml
sudo systemctl restart mihomo
```

---

## 六、安全加固（已完成）

| 项目 | 状态 |
|------|------|
| `secret` | 已改为随机强密码 |
| `allow-lan` | 已改为`false` |
| 配置文件权限 | 已设为`600` |

---

## 七、未来节点扩展指南

当添加新节点时，只需修改`proxies`和`AUTO`组：

```yaml
proxies:
  - name: "Azure-VLESS-Reality"
    # ...现有节点
  - name: "hk-node"
    # ...新节点
  - name: "jp-node"
    # ...新节点

proxy-groups:
  - name: "AUTO"
    type: url-test
    proxies:
      - "Azure-VLESS-Reality"
      - "hk-node"
      - "jp-node"
```

规则部分**无需任何修改**，AI/STREAM/PROXY策略组会自动包含新节点。

---

*文档版本：v2.0*  
*适用环境：mihomo (Clash.Meta) + TUN system 栈 + VLESS/Trojan 节点 + 移动笔记本*
