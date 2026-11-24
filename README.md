# sing-box 多协议与中转管理脚本

基于 [fscarmen/sing-box](https://github.com/fscarmen/sing-box)（`v1.3.31`）二次开发优化的 `sing-box` 一键部署与运维脚本，遵循 GNU GPL v3 协议。

相比原版的核心改进：
- **纯净轻量化**：彻底移除过时的内置 WARP 链式代理解锁与运行统计上报，服务端默认纯直连出站。
- **中转 / 落地一体化**：内置 `sb -o` 中转出口管理，支持直接粘贴落地机分享链接、一键全局中转、Geosite 规则集分流（内置 DMM / AI 等预设）、延迟测速与 `urltest` 故障自动切换；新增 `sb -p` 极简落地机模式。
- **纯 IPv6 深度修复**：修复并发镜像探测竞态误杀、版本号校验污染、多代理自动故障转移及本地 `sb` 离线缓存兜底。
- **运维与安全增强**：内置 `sb -i` 实时连接/流量/日志排查、`sb -m` 配置一键备份迁移、`sb -d` 出站 IP/DNS 策略切换与私有 IP/BT 防滥用拦截；修复多项原版脚本 Bug。

---

## 1. 快速安装

- **常规双栈 / IPv4 主机：**
  ```bash
  bash <(wget -qO- https://raw.githubusercontent.com/huai3/sing-box/main/sing-box.sh)
  ```

- **纯 IPv6 主机（无法直连 GitHub 时）：**
  ```bash
  bash <(wget -qO- https://gh-proxy.com/https://raw.githubusercontent.com/huai3/sing-box/main/sing-box.sh)
  ```

- **极简落地机安装（仅装轻量协议，不装 Nginx / Argo / 订阅）：**
  ```bash
  bash <(wget -qO- https://raw.githubusercontent.com/huai3/sing-box/main/sing-box.sh) -p
  ```

- **安装完成后随时呼出菜单：**
  ```bash
  sb
  ```

> 支持系统：Debian / Ubuntu / CentOS / Rocky / Alma / Fedora / Alpine（支持 systemd 与 OpenRC，兼容 amd64 / arm64 / s390x）。

---

## 2. 常用快捷命令

| 命令 | 说明 |
| :--- | :--- |
| `sb` | 打开主管理菜单 |
| `sb -n` | 查看所有节点链接、订阅地址与二维码 |
| `sb -o` | **中转 / 落地出口管理**（粘贴链接、全局中转、Geosite/域名/入站分流、测速、故障切换） |
| `sb -i` | **实时监控与排障**（累计流量、活跃连接命中规则与出口、查看/清空日志） |
| `sb -d` | **修改节点与系统配置**（端口、SNI、UUID、出站 IP/DNS 策略、私有 IP/BT 防滥用、静态/动态订阅等） |
| `sb -m` | **配置一键备份与恢复迁移**（导出/导入 `.tar.gz`） |
| `sb -p` | **极简落地机安装模式**（仅装 VLESS-Reality / SS-2022 / AnyTLS / Hysteria2） |
| `sb -r` | 增删已安装的入站协议 |
| `sb -s` | 启动 / 停止 `sing-box` |
| `sb -a` | 启动 / 停止 Argo 隧道 |
| `sb -t` | 切换 Argo 临时 / 固定隧道 |
| `sb -v` | 升级或切换 `sing-box` / `cloudflared` 版本 |
| `sb -u` | 卸载全部组件 |

---

## 3. 中转机 / 落地机典型用法

### 场景 A：搭建极简落地机
在落地机执行 `bash <(wget -qO- https://raw.githubusercontent.com/huai3/sing-box/main/sing-box.sh) -p`，按提示选择轻量协议（推荐 `VLESS + Reality` 或 `Shadowsocks 2022`），装完复制输出的分享链接即可。

### 场景 B：中转机全局转发到落地机
在中转机运行 `sb -o`：
1. 选择 `1`（添加落地机出口），直接粘贴落地机分享链接（支持 `vless://`、`ss://`、`anytls://`、`hy2://`、`tuic://`、`trojan://`、`socks5://`、`http://` 或原生 sing-box outbound JSON）。
2. 提示「是否立即设为全局中转出口」时直接回车即可生效。

### 场景 C：按网站分流（如 DMM 走日本、AI 走家宽，其余直连）
在中转机运行 `sb -o`：
1. 选择 `1`，依次粘贴日本落地机（命名为 `jp`）和家宽落地机（命名为 `ai-home`），提示是否设全局时输入 `n` 保持默认直连。
2. 选择 `3`（按 Geosite 规则集分流）：
   - 选出口 `jp` → 选预设 `1. DMM (geosite-dmm)`；
   - 选出口 `ai-home` → 选预设 `2. AI 国际全集 (geosite-category-ai-!cn)`；
   - 也支持选 `6` 自定义输入任意 `geosite`（如 `netflix,disney,bahamut,abema`），远程 `.srs` 自动适配 `GH_PROXY`。
3. 进阶操作：
   - 选 `6` 可**单条删除**某条分流规则；
   - 选 `7` 可**测试各落地机延迟与连通性**；
   - 选 `8` 可创建 **`urltest` 故障自动切换组**（主落地断线自动切备用或回退直连）；
   - 运行 `sb -i` 可实时查看每条连接命中了哪条规则、走了哪个出口。

> 所有中转配置写入独立的 `08_relay_outbound.json` 与 `09_relay_route.json`，升级/重置基础配置时不会被覆盖；每次修改均自动执行 `sing-box check`，校验失败自动回滚。

---

## 4. 支持的入站协议与订阅

- **入站协议**：VLESS + Reality（TCP / H2 / gRPC）、Hysteria2（支持端口跳跃与 Realm）、TUIC v5、Shadowsocks-2022、Trojan、AnyTLS、ShadowTLS v3、VMess + WS、VLESS + WS + TLS（支持 Cloudflare CDN 与 Argo 隧道）、NaiveProxy、Snell v6。
- **订阅输出**：支持 V2rayN / Shadowrocket / Clash (Mihomo) / sing-box 客户端订阅与二维码；支持在 `sb -d` 中开启**动态订阅**（按请求 Host 实时渲染节点地址，适配多 IP 与 IPv6 地址池）。

---

## 5. 无人值守安装

支持通过 [config.conf](config.conf) 或命令行参数批量部署，完整参数说明见 [config.conf](config.conf)。

- **通过配置文件安装：**
  ```bash
  wget -O config.conf https://raw.githubusercontent.com/huai3/sing-box/main/config.conf
  # 按需编辑 config.conf 后执行：
  bash <(wget -qO- https://raw.githubusercontent.com/huai3/sing-box/main/sing-box.sh) -f config.conf
  ```

- **命令行直接带中转出口安装示例：**
  ```bash
  bash <(wget -qO- https://raw.githubusercontent.com/huai3/sing-box/main/sing-box.sh) \
    --CHOOSE_PROTOCOLS bf --START_PORT 20000 \
    --RELAY_URI "vless://..." --RELAY_MODE global
  ```
  > 如只需对特定站点走落地机，可传 `--RELAY_MODE none --RELAY_GEOSITES "dmm,ai"` 或 `--RELAY_DOMAINS "openai.com,netflix.com"`。

---

## 6. 核心目录结构

所有配置位于 `/etc/sing-box/`，采用 `sing-box -C /etc/sing-box/conf/` 多文件自动合并：

```text
/etc/sing-box/
├── sing-box                     # 核心二进制
├── jq                           # JSON 处理工具
├── sb_local.sh                  # sb 命令本地离线兜底脚本
├── logs/box.log                 # 运行日志
├── subscribe/                   # 静态订阅文件与二维码
└── conf/
    ├── 00_log.json              # 日志配置
    ├── 01_outbounds.json        # 默认 direct 出站（含网卡绑定）
    ├── 03_route.json            # 基础路由（sniff）
    ├── 04_experimental.json     # cache_file 与本地 clash_api
    ├── 05_dns.json              # DNS 与出站 IP 策略
    ├── 08_security_route.json   # 安全防滥用拦截（私有 IP / BT，sb -d 管理）
    ├── 08_relay_outbound.json   # 中转落地机出口与 urltest 组（sb -o 管理）
    ├── 09_relay_route.json      # 中转分流规则与 rule_set（sb -o 管理）
    └── 10_*_inbounds.json       # 各协议入站配置
```

---

## 7. 更新日志

<details>
<summary>点击展开 / 收起更新历史（Fork 后的改动）</summary>

- **2026-11-21**
  - 同步上游 `v1.3.31` 修订中的 3 项优化：`check_system_ip()` 会话内缓存、`check_install()` 统一以 `nginx.conf` 判定订阅状态、动态订阅模板优先复用本地缓存。
  - 新增 `sb -o` **中转 / 落地出口管理**：支持粘贴分享链接或原生 JSON、全局中转、Geosite 规则集分流（内置 DMM / AI / Netflix / Disney+ / TikTok 预设及自定义）、按域名/入站分流、单条删除规则、延迟测速与 `urltest` 故障自动切换组；支持 `config.conf` / CLI 无交互参数。
  - 新增 `sb -i` **实时监控与排障**（基于 `clash_api` 查看流量、活跃连接命中规则及日志）。
  - 新增 `sb -m` **配置一键备份与恢复迁移**。
  - `sb -d` 新增 **出站 IP / DNS 策略一键切换** 与 **安全防滥用拦截开关**（私有 IP / BitTorrent）。
  - 新增 `sb -p` **极简落地机安装预设**。
- **2026-11-18**
  - Fork 自 `fscarmen/sing-box`（`v1.3.31`），移除全部内置 WARP 链式代理解锁 ChatGPT 与统计上报逻辑。
  - 修复纯 IPv6 主机下 GitHub 代理并发探测竞态（`wait -n` 误杀可用双栈镜像）、版本号校验污染与内核下载失败问题，增加多代理自动故障转移及 `sb` 本地离线兜底。
  - 修复 `sb -d` 误替换二进制文件、修改起始端口冲突、Reality 私钥外发第三方 API 改本地 OpenSSL 推导、`config.conf` 无交互解析、Alpine BusyBox `base64` 兼容、Docker 复用已有 Argo Tunnel 等多项 Bug。

</details>

---

## 8. 开源协议与致谢

- 本项目基于 [fscarmen/sing-box](https://github.com/fscarmen/sing-box) 二次开发，感谢原作者的开源贡献。
- 本项目严格遵守 GNU GPL v3 许可证，详见 [LICENSE](LICENSE)。
