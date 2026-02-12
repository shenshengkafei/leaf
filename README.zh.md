# Leaf

Leaf 是一个轻量且快速的代理工具。

## 目录

- [Downloads](#downloads)
- [iOS TestFlight](#ios-testflight)
- [CLI 命令行参数](#cli-命令行参数)
- [conf](#conf)
  * [General](#general)
  * [Proxy](#proxy-1)
  * [Proxy Group](#proxy-group)
  * [Rule](#rule)
  * [Host](#host)
  * [Env](#env)
  * [Certificate](#certificate)
- [json](#json)
  * [Log](#log)
  * [DNS](#dns)
  * [inbounds](#inbounds)
    + [http](#http)
    + [socks](#socks)
    + [shadowsocks (inbound)](#shadowsocks-inbound)
    + [trojan (inbound)](#trojan-inbound)
    + [ws](#ws)
    + [tls (inbound)](#tls-inbound)
    + [quic (inbound)](#quic-inbound)
    + [amux](#amux)
    + [chain](#chain)
    + [tun](#tun-inbound)
    + [hc](#hc)
    + [cat](#cat)
  * [outbounds](#outbounds)
    + [direct](#direct)
    + [drop](#drop)
    + [redirect](#redirect)
    + [tls](#tls)
    + [ws](#ws-1)
    + [amux](#amux-1)
    + [h2](#h2)
    + [quic](#quic)
    + [obfs](#obfs)
    + [shadowsocks](#shadowsocks)
    + [vmess](#vmess)
    + [trojan](#trojan)
    + [socks](#socks-1)
    + [chain](#chain-1)
    + [failover](#failover)
    + [tryall](#tryall)
    + [static](#static)
    + [select](#select)
    + [random](#random)
    + [retry](#retry)
    + [plugin](#plugin)
  * [Router / Rules](#router--rules)
    + [domain](#domain)
    + [domainSuffix](#domainsuffix)
    + [domainKeyword](#domainkeyword)
    + [ip](#ip)
    + [geoip](#geoip)
    + [external](#external)
    + [portRange](#portrange)
    + [network](#network)
    + [inboundTag](#inboundtag)
    + [processName](#processname)
- [环境变量](#环境变量)
- [Advanced Features](#advanced-features)
  * [TUN inbound](#tun-inbound-1)

## Downloads

https://github.com/eycorsican/leaf/releases

## iOS TestFlight
iOS TF 测试公开链接：https://testflight.apple.com/join/std0FFCS

## CLI 命令行参数

```
leaf [OPTIONS]

OPTIONS:
  -c, --config <config>          配置文件路径 (默认: "config.conf")
      --auto-reload              配置文件变更时自动重载
      --single-thread            单线程模式运行
      --thread-stack-size <size> 工作线程栈大小 (默认: release 256KB, debug 2MB)
  -T, --test                     测试配置文件并退出
  -t, --test-outbound <tag>      测试指定 outbound 的连通性
  -d, --test-outbound-timeout <seconds>
                                 outbound 连通性测试超时时间，单位秒 (默认: 4)
  -b, --boundif <interface>      绑定网络接口，设置 OUTBOUND_INTERFACE 环境变量
                                 (仅 macOS, Linux, Windows)
  -V, --version                  打印版本信息
```

示例：

```sh
# 使用指定配置文件运行
leaf -c /path/to/config.json

# 测试配置文件有效性
leaf -T -c config.conf

# 测试某个 outbound 的连通性
leaf -t my_proxy -c config.conf

# 配置文件变更时自动重载
leaf --auto-reload -c config.conf

# 绑定到指定网络接口
leaf -b en0 -c config.conf
```

## conf

### General

```ini
[General]
loglevel = info
logoutput = /path/to/leaf.log
dns-server = 114.114.114.114, 223.5.5.5
dns-interface = eth0
routing-domain-resolve = true
always-real-ip = tracker, apple.com
always-fake-ip = *.example.com

# Local HTTP CONNECT proxy
http-interface = 127.0.0.1
http-port = 1087

# "interface" 和 "port" 是 http-interface 和 http-port 的别名
# interface = 127.0.0.1
# port = 1087

# Local SOCKS5 proxy with UDP Associate support
socks-interface = 127.0.0.1
socks-port = 1086

# API 服务器
api-interface = 127.0.0.1
api-port = 9991

# TUN 模式 (macOS 和 Linux)
# 方式 1: auto 模式
tun = auto
# 方式 2: 手动模式
# tun = utun8, 10.10.0.2, 255.255.255.0, 10.10.0.1, 1500
# 方式 3: 文件描述符
# tun-fd = 123
```

General 所有设置：

| 设置 | 说明 |
|------|------|
| `loglevel` | 日志级别：`trace`, `debug`, `info`, `warn`, `error` |
| `logoutput` | 日志输出文件路径 |
| `dns-server` | 逗号分隔的 DNS 服务器列表 |
| `dns-interface` | DNS 绑定接口 |
| `routing-domain-resolve` | 是否为路由规则匹配解析域名 (`true`/`false`) |
| `always-real-ip` | 始终返回真实 IP 的域名列表（逗号分隔，用于 TUN/NF 模式） |
| `always-fake-ip` | 始终返回伪造 IP 的域名列表（逗号分隔，用于 TUN/NF 模式） |
| `http-interface` / `interface` | HTTP 代理监听地址 |
| `http-port` / `port` | HTTP 代理监听端口 |
| `socks-interface` | SOCKS5 代理监听地址 |
| `socks-port` | SOCKS5 代理监听端口 |
| `api-interface` | API 服务器监听地址 |
| `api-port` | API 服务器监听端口 |
| `tun` | TUN 模式：`auto` 或 `name, address, netmask, gateway, mtu` |
| `tun-fd` | TUN 文件描述符（移动平台使用） |
| `nf` | 网络过滤器（Windows）：`driver_name` 或 `driver_name, path/to/nfapi.dll` |

### Proxy

```ini
[Proxy]
Direct = direct
Reject = reject

# Shadowsocks
SS = ss, 1.2.3.4, 8485, encrypt-method=chacha20-ietf-poly1305, password=123456

# Shadowsocks with prefix
SSPrefix = ss, 1.2.3.4, 8485, encrypt-method=chacha20-ietf-poly1305, password=123456, prefix=GET /

# Shadowsocks with simple-obfs
SSObfs = ss, 1.2.3.4, 8485, encrypt-method=chacha20-ietf-poly1305, password=123456, obfs=http, obfs-host=www.example.com, obfs-path=/

# VMess
VMess = vmess, my.domain.com, 8001, username=0eb5486e-e1b5-49c5-aa75-d15e54dfac9d

# VMess over WebSocket over TLS (TLS + WebSocket + VMess)
VMessWSS = vmess, my.domain.com, 443, username=0eb5486e-e1b5-49c5-aa75-d15e54dfac9d, ws=true, tls=true, ws-path=/v2

# Trojan (with TLS)
Trojan = trojan, 4.3.2.1, 443, password=123456, sni=www.domain.com

# Trojan 使用自定义 TLS 证书
TrojanCert = trojan, 4.3.2.1, 443, password=123456, sni=www.domain.com, tls-cert=my_cert

# Trojan 跳过 TLS 验证
TrojanInsecure = trojan, 4.3.2.1, 443, password=123456, sni=www.domain.com, tls-insecure=true

# Trojan over WebSocket over TLS (TLS + WebSocket + Trojan)
TrojanWS = trojan, 4.3.2.1, 443, password=123456, sni=www.domain.com, ws=true, ws-path=/abc

# Trojan over QUIC
TrojanQUIC = trojan, 4.3.2.1, 443, password=123456, sni=www.domain.com, quic=true

# Trojan over amux streams which use WebSocket over TLS as the underlying connection (TLS + WebSocket + amux + Trojan)
tls-ws-amux-trojan = trojan, www.domain.com, 443, password=112358, tls=true, ws=true, ws-path=/amux, amux=true
tls-ws-amux-trojan2 = trojan, 1.0.0.1, 443, password=123456, sni=www.domain.com, ws=true, ws-path=/amux, ws-host=www.domain.com, amux=true, amux-max=16, amux-con=1, amux-max-recv=4194304, amux-max-lifetime=1800

# SOCKS5 outbound（支持用户密码认证）
Socks = socks, 1.2.3.4, 1080
SocksAuth = socks, 1.2.3.4, 1080, username=user, password=pass

# Redirect
Redirect = redirect, 1.2.3.4, 8080
```

Proxy 所有可用参数：

| 参数 | 适用协议 | 说明 |
|------|---------|------|
| `encrypt-method` | ss, vmess | 加密方式（默认: `chacha20-ietf-poly1305`） |
| `password` | ss, trojan, socks | 密码 |
| `prefix` | ss | Shadowsocks 前缀字节 |
| `username` | vmess, socks | UUID (vmess) 或用户名 (socks) |
| `obfs` | ss | 混淆类型：`http`, `tls` |
| `obfs-host` | ss | 混淆主机名 |
| `obfs-path` | ss | 混淆路径（默认: `/`） |
| `ws` | trojan, vmess | 启用 WebSocket (`true`/`false`) |
| `ws-path` | trojan, vmess | WebSocket 路径（默认: `/`） |
| `ws-host` | trojan, vmess | WebSocket Host 头 |
| `tls` | trojan, vmess | 启用 TLS (`true`/`false`) |
| `tls-cert` | trojan, vmess | TLS 证书（文件路径或 Certificate 节名称） |
| `tls-insecure` | trojan, vmess | 跳过 TLS 验证 (`true`/`false`) |
| `sni` | trojan, vmess | TLS 服务器名称 |
| `amux` | trojan, vmess | 启用 AMux 多路复用 (`true`/`false`) |
| `amux-max` | trojan, vmess | 单个 AMux 连接最多流数量（默认: `8`） |
| `amux-con` | trojan, vmess | AMux 并发流数量（默认: `2`） |
| `amux-max-recv` | trojan, vmess | AMux 单连接最大接收字节数（默认: `0` = 无限） |
| `amux-max-lifetime` | trojan, vmess | AMux 单连接最大生命周期秒数（默认: `0` = 无限） |
| `quic` | trojan, vmess | 启用 QUIC 传输 (`true`/`false`) |
| `interface` | 所有 | 出站连接绑定地址 |

### Proxy Group

```ini
[Proxy Group]
# failover: 按顺序尝试，直到找到可用的 outbound
Failover = failover, Trojan, VMessWSS, SS, health-check=true, check-interval=600, fail-timeout=5, failover=true

# fallback 等效于 failover
Fallback = fallback, Trojan, VMessWSS, SS, check-interval=600, fail-timeout=5

# url-test 等效于 failover=false 的 failover（仅按健康检查排序，不 failover）
UrlTest = url-test, Trojan, VMessWSS, SS, check-interval=600, fail-timeout=5

# failover 带 fallback cache
FailoverCache = failover, Direct, Trojan, SS, health-check=false, failover=true, fallback-cache=true, cache-size=256, cache-timeout=60

# failover 带高级健康检查设置
FailoverAdv = failover, Trojan, VMessWSS, health-check=true, check-interval=300, fail-timeout=4, health-check-timeout=6, health-check-delay=200, health-check-active=900, health-check-on-start=true, health-check-wait=true, health-check-attempts=2, health-check-success-percentage=50

# failover 带 last resort 和偏好 outbound
FailoverLR = failover, Trojan, VMessWSS, health-check=true, last-resort=Direct, health-check-prefers=Trojan:VMessWSS

# tryall: 同时向所有 outbound 发起请求，选取最快的
Tryall = tryall, Trojan, VMessWSS, delay-base=0

# static: 静态选择（random 或 shuffle）
Random = static, Trojan, VMessWSS, method=random
Shuffle = static, Trojan, VMessWSS, method=shuffle

# select: 动态选择（通过 API 控制）
MySelect = select, Trojan, VMessWSS, Direct

# chain: 代理链
Chain = chain, SS, Trojan
```

Failover 所有参数：

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `health-check` | `true` | 启用健康检查 |
| `check-interval` | `300` | 健康检查间隔（秒） |
| `fail-timeout` | `4` | 握手超时（秒），包括 TCP 握手及相应代理协议握手的时间 |
| `failover` | `true` | 失败时尝试下一个；`false` 则只使用最优的 |
| `fallback-cache` | `false` | 缓存成功的 fallback outbound |
| `cache-size` | `256` | fallback cache 大小 |
| `cache-timeout` | `60` | fallback cache 缓存时间（分钟） |
| `last-resort` | - | 最后手段 outbound 的 tag |
| `health-check-timeout` | `6` | 健康检查超时（秒） |
| `health-check-delay` | `200` | 各 outbound 健康检查间延迟（毫秒） |
| `health-check-active` | `900` | 仅检查在此秒数内有活动的 outbound |
| `health-check-prefers` | - | 偏好的 outbound tag 列表（冒号分隔） |
| `health-check-on-start` | `false` | 启动时立即执行健康检查 |
| `health-check-wait` | `false` | 等待首次健康检查完成后再提供服务 |
| `health-check-attempts` | `1` | 每次健康检查的尝试次数 |
| `health-check-success-percentage` | `50` | 健康检查要求的成功百分比 |

### Rule

```ini
[Rule]
# 执行文件目录当中必需有 `site.dat` 文件
EXTERNAL, site:category-ads-all, Reject

# 也可以指定 `dat` 文件所在绝对路径，不支持相对路径
EXTERNAL, site:/tmp/geosite.dat:category-ads-all, Reject

IP-CIDR, 8.8.8.8/32, Fallback
DOMAIN, www.google.com, Fallback
DOMAIN-SUFFIX, google.com, Fallback
DOMAIN-KEYWORD, google, Fallback

# 等效于 EXTERNAL, mmdb:us, Fallback
GEOIP, us, Fallback

EXTERNAL, site:geolocation-!cn, Fallback

# 执行文件目录当中必需有 `geo.mmdb` 文件
EXTERNAL, mmdb:us, Fallback

# 端口范围匹配
PORT-RANGE, 80-80, Direct

# 网络类型匹配
NETWORK, TCP, Fallback

# 按 inbound tag 匹配
INBOUND-TAG, socks, Fallback

# 按进程名匹配（需要 rule-process-name feature）
PROCESS-NAME, curl, Fallback

FINAL, Direct
```

规则类型：

| 规则 | 说明 |
|------|------|
| `DOMAIN` | 匹配整个域名 |
| `DOMAIN-SUFFIX` | 匹配子域名（`google.com` 匹配 `www.google.com`，但不匹配 `wwwgoogle.com`） |
| `DOMAIN-KEYWORD` | 匹配域名关键字 |
| `IP-CIDR` | 匹配 IP 或 IP-CIDR |
| `GEOIP` | 匹配 GeoIP 国家代码（需要 `geo.mmdb` 文件） |
| `EXTERNAL` | 外部规则文件（`site:TAG` 或 `mmdb:TAG`） |
| `PORT-RANGE` | 匹配端口范围（如 `80-80`, `1000-2000`） |
| `NETWORK` | 匹配网络类型：`TCP` 或 `UDP` |
| `INBOUND-TAG` | 按 inbound tag 匹配 |
| `PROCESS-NAME` | 按进程名匹配（需要 `rule-process-name` feature） |
| `FINAL` | 默认规则，必须放在最后 |

### Host

对指定域名返回一个或多个静态 IP：

```ini
[Host]
example.com = 192.168.0.1, 192.168.0.2
```

作为 `hosts` 的使用例子，以下两个配置在效果上是相同的（因为用 json 配置会很长，这里用 conf 表达）：

```ini
[Proxy]
Proxy = trojan, www.domain.com, 443, password=123456, ws=true, ws-path=/abc
[Host]
www.domain.com = 1.2.3.4
```

```ini
[Proxy]
Proxy = trojan, 1.2.3.4, 443, password=123456, ws=true, ws-path=/abc, sni=www.domain.com
```

而 `hosts` 还可以指定多个 IP：

```ini
[Host]
www.domain.com = 1.2.3.4, 5.6.7.8
```

### Env

在配置文件中设置环境变量，这些变量在其它配置节解析之前生效。

```ini
[Env]
ENABLE_IPV6 = true
LOG_NO_COLOR = true
TLS_DOMAIN_SNIFFING = true
OUTBOUND_INTERFACE = en0
```

### Certificate

内联 TLS 证书，可以在 `[Proxy]` 节通过 `tls-cert` 参数按名称引用。

```ini
[Certificate.my_cert]
-----BEGIN CERTIFICATE-----
MIICxxxxxxxx...
-----END CERTIFICATE-----

[Certificate.another_cert]
-----BEGIN CERTIFICATE-----
MIICyyyyyyyy...
-----END CERTIFICATE-----
```

在 `[Proxy]` 中使用：

```ini
[Proxy]
Trojan = trojan, 4.3.2.1, 443, password=123456, sni=www.domain.com, tls-cert=my_cert
```

---

在 [AppStore](https://apps.apple.com/us/app/leaf-lightweight-proxy/id1534109007) 或 [TestFlight](https://testflight.apple.com/join/std0FFCS) （都可以免费下载到）上的 Leaf 中，版本 `1.1 (8)` 及以上，`conf` 格式除了以上设置以外还支持一个 `[On Demand]` 配置，这是完全是一个 iOS 方面的功能，跟本 leaf 项目关系不大，它不涉及任何 Rust 代码，但为了方便查看也在这写下。

下面规则表示连接 `OpenWrt` WiFi 信号时断开 VPN，其它任何情况都连着 VPN，典型的使用场景是 OpenWrt 是一个有透明代理的无线信号：

```ini
[On Demand]
# 表示如果当前连接到 wifi 且 ssid 名为 OpenWrt，则断开 VPN
DISCONNECT, ssid=OpenWrt, interface-type=wifi
# 无条件地连接 VPN
CONNECT
```

规则有两种 `CONNECT` 和 `DISCONNECT` ，匹配条件支持两种 `ssid` 和 `interface-type`，`ssid` 可以是以 `:` 分隔的 ssid 名称列表，`interface-type` 只能是以下 3 个值中的一个：`wifi`, `cellular`, `any`。规则不带任何匹配条件表示无条件执行。

## json

JSON 配置文件目前不考虑兼容性，每个版本都可能会变。

顶层结构：

```json
{
    "log": { ... },
    "dns": { ... },
    "inbounds": [ ... ],
    "outbounds": [ ... ],
    "router": { ... }
}
```

完整示例：

```json
{
    "log": {
        "level": "info",
        "output": "console"
    },
    "dns": {
        "servers": [
            "1.1.1.1",
            "8.8.8.8"
        ],
        "hosts": {
            "example.com": [
                "192.168.0.1",
                "192.168.0.2"
            ],
            "server.com": [
                "192.168.0.3"
            ]
        }
    },
    "inbounds": [
        {
            "address": "127.0.0.1",
            "port": 1087,
            "protocol": "http"
        },
        {
            "address": "127.0.0.1",
            "port": 1086,
            "protocol": "socks"
        }
    ],
    "outbounds": [
        {
            "protocol": "failover",
            "settings": {
                "actors": [
                    "vmess_out",
                    "trojan_out"
                ]
            },
            "tag": "failover_out"
        },
        {
            "protocol": "chain",
            "settings": {
                "actors": [
                    "vmess_tls",
                    "vmess_ws",
                    "vmess"
                ]
            },
            "tag": "vmess_out"
        },
        {
            "protocol": "tls",
            "tag": "vmess_tls"
        },
        {
            "protocol": "ws",
            "settings": {
                "path": "/v2"
            },
            "tag": "vmess_ws"
        },
        {
            "protocol": "vmess",
            "settings": {
                "address": "server.com",
                "port": 443,
                "uuid": "89ee4e17-aaad-49f6-91c4-6ea5990206bd"
            },
            "tag": "vmess"
        },
        {
            "protocol": "chain",
            "settings": {
                "actors": [
                    "trojan_tls",
                    "trojan"
                ]
            },
            "tag": "trojan_out"
        },
        {
            "protocol": "tls",
            "tag": "trojan_tls"
        },
        {
            "protocol": "trojan",
            "settings": {
                "address": "server.com",
                "password": "112358",
                "port": 443
            },
            "tag": "trojan"
        },
        {
            "protocol": "shadowsocks",
            "settings": {
                "address": "x.x.x.x",
                "method": "chacha20-ietf-poly1305",
                "password": "123456",
                "port": 8389
            },
            "tag": "shadowsocks_out"
        },
        {
            "protocol": "socks",
            "settings": {
                "address": "x.x.x.x",
                "port": 1080
            },
            "tag": "socks_out"
        },
        {
            "protocol": "direct",
            "tag": "direct_out"
        },
        {
            "protocol": "drop",
            "tag": "drop_out"
        }
    ],
    "router": {
        "domainResolve": false,
        "rules": [
            {
                "ip": [
                    "8.8.8.8",
                    "8.8.4.4"
                ],
                "target": "failover_out"
            },
            {
                "domain": [
                    "www.google.com"
                ],
                "target": "failover_out"
            },
            {
                "domainSuffix": [
                    "google.com"
                ],
                "target": "failover_out"
            },
            {
                "domainKeyword": [
                    "google"
                ],
                "target": "failover_out"
            },
            {
                "external": [
                    "site:cn"
                ],
                "target": "direct_out"
            },
            {
                "external": [
                    "mmdb:cn"
                ],
                "target": "direct_out"
            }
        ]
    }
}
```

## Log

```json
"log": {
    "level": "info",
    "output": "/path/to/leaf.log"
}
```

- `level` 可以是 `trace`, `debug`, `info`, `warn`, `error`, `none`
- `output` 是日志文件路径，或 `"console"` 表示输出到标准输出

## DNS

```json
"dns": {
    "servers": [
        "114.114.114.114",
        "1.1.1.1"
    ],
    "hosts": {
        "example.com": [
            "192.168.0.1",
            "192.168.0.2"
        ],
        "server.com": [
            "192.168.0.3"
        ]
    }
}
```

DNS 用于 `direct` outbound 请求的域名解析，以及其它 outbound 中代理服务器地址的解析（如果代理服务器地址是 IP，则不需要解析）。`servers` 是 DNS 服务器列表，`hosts` 是静态 IP。


作为 `hosts` 的使用例子，以下两个配置在效果上是相同的（因为用 json 配置会很长，这里用 conf 表达）：

```ini
[Proxy]
Proxy = trojan, www.domain.com, 443, password=123456, ws=true, ws-path=/abc
[Host]
www.domain.com = 1.2.3.4
```

```ini
[Proxy]
Proxy = trojan, 1.2.3.4, 443, password=123456, ws=true, ws-path=/abc, sni=www.domain.com
```

而 `hosts` 还可以指定多个 IP：

```ini
[Host]
www.domain.com = 1.2.3.4, 5.6.7.8
```

## inbounds

```json
"inbounds": [
    {
        ...
    },
    {
        ...
    }
]
```

inbounds 是一个数组，每一项可以是以下：

### http

```json
{
    "protocol": "http",
    "address": "127.0.0.1",
    "port": 1087
}
```

支持 HTTP Connect。

### socks

```json
{
    "protocol": "socks",
    "address": "127.0.0.1",
    "port": 1086
}
```

默认支持 UDP。

### shadowsocks (inbound)

```json
{
    "protocol": "shadowsocks",
    "tag": "ss_in",
    "address": "0.0.0.0",
    "port": 8388,
    "settings": {
        "method": "chacha20-ietf-poly1305",
        "password": "123456"
    }
}
```

### trojan (inbound)

```json
{
    "protocol": "trojan",
    "tag": "trojan_in",
    "address": "0.0.0.0",
    "port": 10086,
    "settings": {
        "passwords": ["password1", "password2"]
    }
}
```

trojan inbound 支持多个密码。

### ws

WebSocket 传输，一般在 `chain` 叠加到其它代理协议上。

```json
{
    "protocol": "ws",
    "tag": "ws_in",
    "settings": {
        "path": "/abc"
    }
}
```

### tls (inbound)

TLS 传输，一般在 `chain` 叠加到其它代理协议上。

```json
{
    "protocol": "tls",
    "tag": "tls_in",
    "settings": {
        "certificate": "/path/to/cert.pem",
        "certificateKey": "/path/to/key.pem"
    }
}
```

`certificate` 也支持内联 PEM 证书内容。

### quic (inbound)

```json
{
    "protocol": "quic",
    "tag": "quic_in",
    "address": "0.0.0.0",
    "port": 443,
    "settings": {
        "certificate": "/path/to/cert.pem",
        "certificateKey": "/path/to/key.pem",
        "alpn": ["h3"]
    }
}
```

### amux

`amux` 多路复用传输，可以在一个可靠的连接上建立多个可靠流传输。

**`amux` 目前不提供版本间兼容。**

```json
{
    "protocol": "amux",
    "tag": "amux_in",
    "settings": {
        "actors": [
             "tls",
             "ws"
        ]
    }
}
```

- `actors` 指定底层传输，空值表示用 TCP

### chain

`chain` 可以对多个协议进行叠加。

```json
{
    "protocol": "chain",
    "address": "127.0.0.1",
    "port": 10086,
    "settings": {
        "actors": [
            "ws_out",
            "trojan_out"
        ]
    }
}
```

例如这是一个 WebSocket + Trojan 配置：

```json
"inbounds": [
    {
        "protocol": "chain",
        "tag": "ws_trojan_in",
        "address": "127.0.0.1",
        "port": 4003,
        "settings": {
            "actors": [
                "ws_in",
                "trojan_in"
            ]
        }
    },
    {
        "protocol": "ws",
        "tag": "ws_in",
        "settings": {
            "path": "/abc"
        }
    },
    {
        "protocol": "trojan",
        "tag": "trojan_in",
        "settings": {
            "passwords": ["12345"]
        }
    }
]
```

注意上面配置示例没有 TLS，一般可以交给 nginx 来处理。

### TUN inbound

在 macOS、Linux、iOS 和 Android 上支持 TUN inbound：

```json
{
    "protocol": "tun",
    "tag": "tun_in",
    "settings": {
        "auto": true,
        "name": "utun8",
        "address": "10.10.0.2",
        "netmask": "255.255.255.0",
        "gateway": "10.10.0.1",
        "mtu": 1500,
        "fakeDnsInclude": ["google"],
        "fakeDnsExclude": ["*.local"]
    }
}
```

| 参数 | 说明 |
|------|------|
| `auto` | 自动配置 TUN 接口 |
| `fd` | 文件描述符（移动平台使用，设置后忽略 name/address 等） |
| `name` | 接口名（macOS 用 `utun*`，Linux 用 `tun*`） |
| `address` | TUN 接口地址 |
| `gateway` | TUN 网关地址 |
| `netmask` | TUN 子网掩码 |
| `mtu` | MTU（默认: 1500） |
| `fakeDnsInclude` | 关键字列表：只有匹配的域名会返回伪造 IP |
| `fakeDnsExclude` | 关键字列表：匹配的域名不会返回伪造 IP |

`fakeDnsInclude` 和 `fakeDnsExclude` 只能二选一。

### hc

HTTP 健康检查端点。

```json
{
    "protocol": "hc",
    "tag": "hc_in",
    "address": "127.0.0.1",
    "port": 8080,
    "settings": {
        "path": "/health",
        "response": "OK"
    }
}
```

### cat

转发流量到指定地址。

```json
{
    "protocol": "cat",
    "tag": "cat_in",
    "settings": {
        "network": "tcp",
        "address": "127.0.0.1",
        "port": 8080
    }
}
```

## outbounds

支持常见的代理协议比如 Shadowsocks、VMess、Trojan，以及 TLS 和 WebSocket 传输，另外有多个组合类型的 outbound，其中 `chain` 可以对各种代理和传输协议进行任意组合。

```json
"outbounds": [
    {
        ...
    },
    {
        ...
    }
]
```

outbounds 是一个数组，每一项可以是以下：

### direct

直连出口，请求将从本机直接发往目标，不经任何代理。

```json
{
    "protocol": "direct",
    "tag": "direct_out"
}
```

### drop

拦截请求。

```json
{
    "protocol": "drop",
    "tag": "drop_out"
}
```

### redirect

重定向请求到指定地址。

```json
{
    "protocol": "redirect",
    "tag": "redirect_out",
    "settings": {
        "address": "new.example.com",
        "port": 8080
    }
}
```

### tls

TLS 传输，一般用来叠加到其它代理或传输协议上。

```json
{
    "protocol": "tls",
    "settings": {
        "serverName": "server.com",
        "alpn": ["http/1.1"],
        "certificate": "/path/to/ca.pem",
        "insecure": false
    },
    "tag": "tls_out"
}
```

| 参数 | 说明 |
|------|------|
| `serverName` | TLS SNI，为空会尝试从下层协议获取 |
| `alpn` | ALPN 协议列表 |
| `certificate` | CA 证书文件路径或内联 PEM 证书 |
| `insecure` | 是否跳过服务器证书验证 |

### ws

WebSocket 传输，一般用来叠加到其它代理或传输协议上。

```json
{
    "protocol": "ws",
    "settings": {
        "path": "/v2",
        "headers": {
            "Host": "server.com"
        }
    },
    "tag": "ws_out"
}
```

`headers` 是一个字典，可以包含任意数量的 KV 对。`Host` 不指定的话会尝试从下层协议获取。

### amux

`amux` 多路复用传输，可以在一个可靠的连接上建立多个可靠流传输。

**`amux` 目前不提供版本间兼容。**

```json
{
    "protocol": "amux",
    "settings": {
        "actors": [
             "tls",
             "ws"
        ],
        "address": "tls.server.com",
        "port": 443,
        "maxAccepts": 8,
        "concurrency": 2,
        "max_recv_bytes": 4194304,
        "max_lifetime": 1800
    }
}
```

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `actors` | - | 底层传输，空值表示用 TCP |
| `address` | - | 底层传输的连接地址 |
| `port` | - | 端口 |
| `maxAccepts` | `8` | 单个底层连接最多可建立流的数量 |
| `concurrency` | `2` | 单个底层连接并发流数量 |
| `max_recv_bytes` | `0` | 单个连接最大接收字节数（`0` = 无限） |
| `max_lifetime` | `0` | 单个连接最大生命周期秒数（`0` = 无限） |

`amux` 是一个非常简单的多路复用传输协议，所有流数量的传输都是以 FIFO 方式进行，设计上依赖 `maxAccepts` 和 `concurrency` 两个参数对传输性能进行控制。

### h2

HTTP2 传输，一般需要配合 tls 一起使用，tls 需要配置 h2 作为 alpn。

```json
"outbounds": [
    {
        "protocol": "chain",
        "settings": {
            "actors": [
                "vmess_tls",
                "vmess_h2",
                "vmess"
            ]
        },
        "tag": "vmess_out"
    },
    {
        "protocol": "tls",
        "settings": {
            "serverName": "server.com",
            "alpn": ["h2"]
        },
        "tag": "vmess_tls"
    },
    {
        "protocol": "h2",
        "settings": {
            "host": "server.com",
            "path": "/v2"
        },
        "tag": "vmess_h2"
    },
    {
        "protocol": "vmess",
        "settings": {
            "address": "server.com",
            "port": 443,
            "uuid": "89ee4e17-aaad-49f6-91c4-6ea5990206bd"
        },
        "tag": "vmess"
    }
]
```

### quic

QUIC 传输。

```json
{
    "protocol": "quic",
    "tag": "quic_out",
    "settings": {
        "address": "server.com",
        "port": 443,
        "serverName": "server.com",
        "certificate": "/path/to/ca.pem",
        "alpn": ["h3"]
    }
}
```

### obfs

混淆层，一般配合 Shadowsocks 通过 `chain` 使用。

```json
{
    "protocol": "obfs",
    "tag": "obfs_out",
    "settings": {
        "method": "http",
        "host": "www.example.com",
        "path": "/"
    }
}
```

`method` 支持：`http`, `tls`

使用 chain 组合 obfs + Shadowsocks：

```json
"outbounds": [
    {
        "protocol": "chain",
        "settings": {
            "actors": ["obfs_out", "ss_out"]
        },
        "tag": "obfs_ss_out"
    },
    {
        "protocol": "obfs",
        "settings": {
            "method": "http",
            "host": "www.example.com",
            "path": "/"
        },
        "tag": "obfs_out"
    },
    {
        "protocol": "shadowsocks",
        "settings": {
            "address": "x.x.x.x",
            "method": "chacha20-ietf-poly1305",
            "password": "123456",
            "port": 8389
        },
        "tag": "ss_out"
    }
]
```

### shadowsocks

```json
{
    "protocol": "shadowsocks",
    "settings": {
        "address": "x.x.x.x",
        "method": "chacha20-ietf-poly1305",
        "password": "123456",
        "port": 8389,
        "prefix": "GET /"
    },
    "tag": "shadowsocks_out"
}
```

`method`：
- chacha20-ietf-poly1305
- aes-128-gcm
- aes-256-gcm

`prefix`（可选）：在每个 Shadowsocks 数据包前添加的前缀字节。

### vmess

```json
{
    "protocol": "vmess",
    "settings": {
        "address": "server.com",
        "port": 10086,
        "uuid": "89ee4e17-aaad-49f6-91c4-6ea5990206bd",
        "security": "chacha20-ietf-poly1305"
    },
    "tag": "vmess"
}
```

`security`：
- chacha20-ietf-poly1305
- aes-128-gcm

### trojan

`trojan` outbound 只包含未经 TLS 加密的代理协议，通常还需要利用 `chain` 对其叠加一层 `tls` 才能和正常的 trojan 服务器通讯。

```json
{
    "protocol": "trojan",
    "settings": {
        "address": "server.com",
        "password": "112358",
        "port": 443
    },
    "tag": "trojan_out"
}
```

### socks

```json
{
    "protocol": "socks",
    "settings": {
        "address": "1.2.3.4",
        "port": 1080,
        "username": "user",
        "password": "pass"
    },
    "tag": "socks_out"
}
```

`username` 和 `password` 可选。

### chain

`chain` outbound 可以对任意协议进行叠加，主要用途是在某个代理协议上叠加 tls、ws 等传输，以及配置代理链。

这是一个典型的 TLS + WebSocket + VMess 配置：

```json
"outbounds": [
    {
        "protocol": "chain",
        "settings": {
            "actors": [
                "vmess_tls",
                "vmess_ws",
                "vmess"
            ]
        },
        "tag": "vmess_out"
    },
    {
        "protocol": "tls",
        "tag": "vmess_tls"
    },
    {
        "protocol": "ws",
        "settings": {
            "path": "/v2"
        },
        "tag": "vmess_ws"
    },
    {
        "protocol": "vmess",
        "settings": {
            "address": "server.com",
            "port": 443,
            "uuid": "89ee4e17-aaad-49f6-91c4-6ea5990206bd"
        },
        "tag": "vmess"
    }
]
```

如果有多个服务器，可以配置一个代理链，请求将沿着代理链传输后到达目标：

> 客户端 -> ss1 -> ss2 -> 目标

```json
"outbounds": [
    {
        "protocol": "chain",
        "settings": {
            "actors": [
                "ss1",
                "ss2"
            ]
        },
        "tag": "ss_chain_out"
    },
    {
        "protocol": "shadowsocks",
        "settings": {
            "address": "1.1.1.1",
            "method": "chacha20-ietf-poly1305",
            "password": "123456",
            "port": 1111
        },
        "tag": "ss1"
    },
    {
        "protocol": "shadowsocks",
        "settings": {
            "address": "2.2.2.2",
            "method": "chacha20-ietf-poly1305",
            "password": "123456",
            "port": 2222
        },
        "tag": "ss2"
    }
]
```

### failover

```json
{
    "protocol": "failover",
    "settings": {
        "actors": [
            "vmess_out",
            "trojan_out"
        ],
        "failTimeout": 4,
        "healthCheck": true,
        "checkInterval": 300,
        "healthCheckTimeout": 6,
        "healthCheckDelay": 200,
        "healthCheckActive": 900,
        "healthCheckPrefers": ["vmess_out"],
        "healthCheckOnStart": false,
        "healthCheckWait": false,
        "healthCheckAttempts": 1,
        "healthCheckSuccessPercentage": 50,
        "failover": true,
        "fallbackCache": false,
        "cacheSize": 256,
        "cacheTimeout": 60
    },
    "tag": "failover_out"
}
```

向列表中的 outbound 逐个发送请求，直到找到一个可用的 outbound，可选参数有

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `failTimeout` | `4` | 握手超时（秒），包括 TCP 握手及相应代理协议握手的时间 |
| `healthCheck` | `true` | 如果为 `true`，则对列表中的 outbound 定时做健康检查，并按延迟重新排序 |
| `checkInterval` | `300` | 健康检查间隔（秒） |
| `healthCheckTimeout` | `6` | 健康检查超时（秒） |
| `healthCheckDelay` | `200` | 各 outbound 健康检查间延迟（毫秒） |
| `healthCheckActive` | `900` | 仅检查在此秒数内有活动的 outbound |
| `healthCheckPrefers` | - | 偏好的 outbound tag 列表 |
| `healthCheckOnStart` | `false` | 启动时立即执行健康检查 |
| `healthCheckWait` | `false` | 等待首次健康检查完成后再提供服务 |
| `healthCheckAttempts` | `1` | 每次健康检查的尝试次数 |
| `healthCheckSuccessPercentage` | `50` | 健康检查要求的成功百分比 |
| `failover` | `true` | 如果为 `false`，则只取一个 outbound 发送请求，失败也不会尝试其它 outbound |
| `fallbackCache` | `false` | 如果为 `true`，则对 fallback outbound 的成功请求作记录缓存，后续同样请求直接使用已缓存的 outbound |
| `cacheSize` | `256` | fallback cache 大小 |
| `cacheTimeout` | `60` | fallback cache 缓存时间（分钟） |

`failover` 的 actors 里面可以包含另一个 `failover` outbound，可以实现非常灵活的多级负载分配机制。

`fallbackCache` 功能的初衷是让 `failover` 能够实现自动检测需要代理请求的机制，把一个 `direct` 和任意数量的其它 outbound 放到 `failover` 中，`direct` 放第一位，并禁用 `healthCheck`，启用 `fallbackCache`，那 `failover` 就会先尝试 `direct`，如果失败，自动切换使用其它 outbound，并且记录缓存下来，下一个同样请求直接跳过 `direct` 使用对应 outbound，但有个缺陷是它只能检测 TCP 连接超时或连接错误的请求。所谓 fallback outbound 就是 `failover` actors 里面第一个 outbound 失败后，所用到的后续任意成功的某个 outbound。

### tryall

```json
{
    "protocol": "tryall",
    "settings": {
        "actors": [
            "trojan_out",
            "vmess_out"
        ],
        "delayBase": 0
    },
    "tag": "tryall_out"
}
```

向列表中的所有 outbound 同时发起代理请求，选取握手成功最快的 outbound，可选参数有

- `delayBase` 延时基数，如果大于 0，则代理请求会延迟 delayBase * index 毫秒，index 从 0 起，每个 outbound 递增 1

### static

静态选择，从列表中按 method 选择一个 outbound 发送请求。

```json
{
    "protocol": "static",
    "settings": {
        "actors": [
            "trojan_out",
            "vmess_out"
        ],
        "method": "random"
    },
    "tag": "static_out"
}
```

- `method`：`random`（默认）或 `shuffle`

### select

动态选择器，可以在运行时通过 API 切换 outbound。

```json
{
    "protocol": "select",
    "settings": {
        "actors": [
            "trojan_out",
            "vmess_out",
            "direct_out"
        ]
    },
    "tag": "select_out"
}
```

### random

```json
{
    "protocol": "random",
    "settings": {
        "actors": [
            "trojan_out",
            "vmess_out"
        ]
    },
    "tag": "random"
}
```

从列表中随机选一个 outbound 发送请求。

### retry

```json
{
    "protocol": "retry",
    "settings": {
        "actors": [
            "trojan_out",
            "vmess_out"
        ],
        "attempts": 2
    },
    "tag": "retry"
}
```

可以对 outbound 列表进行多次重试。

### plugin

外部插件 outbound，通过异步 FFI 加载。

```json
{
    "protocol": "plugin",
    "settings": {
        "path": "/path/to/plugin.so",
        "args": "arg1=val1"
    },
    "tag": "plugin_out"
}
```

## Router / Rules

路由规则放在 `router` 对象中，规则决定哪个 outbound 处理请求。`outbounds` 数组中的第一个 outbound 作为默认出口（无规则匹配时使用）。

```json
"router": {
    "domainResolve": true,
    "rules": [
        {
            ...
        },
        {
            ...
        }
    ]
}
```

- `domainResolve`：为 IP 类规则匹配解析域名
- 规则按顺序匹配，第一个匹配的规则生效

`rules` 是一个数组，每一项可以是以下：

### domain

匹配整个域名。

```json
{
    "domain": [
        "www.google.com"
    ],
    "target": "failover_out"
}
```

### domainSuffix

匹配子域名，虽然名字是 `Suffix`，但只匹配子域名，即 `google.com` 匹配 `www.google.com`，但不匹配 `wwwgoogle.com`。

```json
{
    "domainSuffix": [
        "google.com"
    ],
    "target": "failover_out"
}
```

### domainKeyword

匹配域名关键字。

```json
{
    "domainKeyword": [
        "google"
    ],
    "target": "failover_out"
}
```

### ip

匹配 IP 或 IP-CIDR。

```json
{
    "ip": [
        "8.8.8.8/32",
        "8.8.4.4"
    ],
    "target": "failover_out"
}
```

### geoip

可执行文件目录中必需有 `geo.mmdb` 文件存在。

```json
{
    "geoip": [
        "us",
        "jp"
    ],
    "target": "failover_out"
}
```

### external

`external` 规则可以从外部文件加载规则，支持两种格式

```json
{
    "external": [
        "mmdb:us"
    ],
    "target": "failover_out"
}
```

```json
{
    "external": [
        "site:cn"
    ],
    "target": "direct_out"
}
```

#### mmdb

MaxMind 的 mmdb 格式，可以有如下形式：

- `mmdb:TAG` 假设 mmdb 文件存在于可执行文件目录，并且文件名为 `geo.mmdb`
- `mmdb:FILENAME:TAG` 假设 mmdb 文件存在于可执行文件目录，文件名为 `FILENAME`，文件名包含后缀。
- `mmdb:PATH:TAG` 指写 mmdb 文件的绝对路径为 `PATH`，文件名包含后缀。

#### site

V2Ray 的 `dat` 文件格式，可以有如下形式：

- `site:TAG` 同 mmdb，文件名为 `site.dat`
- `site:FILENAME:TAG` 同 mmdb
- `site:PATH:TAG` 同 mmdb

### portRange

匹配端口范围。

```json
{
    "portRange": [
        "80-80",
        "443-443",
        "8000-9000"
    ],
    "target": "direct_out"
}
```

### network

匹配网络类型。

```json
{
    "network": [
        "TCP"
    ],
    "target": "failover_out"
}
```

值可以是 `TCP` 或 `UDP`。

### inboundTag

按 inbound tag 匹配。

```json
{
    "inboundTag": [
        "socks",
        "http"
    ],
    "target": "failover_out"
}
```

### processName

按进程名匹配（需要编译时启用 `rule-process-name` feature）。

```json
{
    "processName": [
        "curl",
        "wget"
    ],
    "target": "failover_out"
}
```

## 环境变量

所有环境变量也可以在 conf 格式的 `[Env]` 节中设置。

### 并发与性能

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `ENDPOINT_TCP_CONCURRENCY` | 1024 (iOS: 45) | 代理出站最大 TCP 并发连接数 |
| `DIRECT_TCP_CONCURRENCY` | 1024 (iOS: 64) | 直连出站最大 TCP 并发连接数 |
| `OUTBOUND_DIAL_CONCURRENCY` | 1 | 出站最大拨号并发数 |
| `LINK_BUFFER_SIZE` | 2 | 连接缓冲区大小（KB） |
| `DATAGRAM_BUFFER_SIZE` | 2 | UDP 数据报缓冲区大小（KB） |
| `DNS_CACHE_SIZE` | 512 (iOS: 64) | DNS 缓存大小 |

### 超时

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `TCP_UPLINK_TIMEOUT` | 10 | 下行 EOF 后上行超时（秒） |
| `TCP_DOWNLINK_TIMEOUT` | 10 | 上行 EOF 后下行超时（秒） |
| `OUTBOUND_DIAL_TIMEOUT` | 4 | 出站连接超时（秒） |
| `INBOUND_ACCEPT_TIMEOUT` | 60 | 入站握手超时（秒） |
| `DNS_TIMEOUT` | 4 | DNS 查询超时（秒） |
| `UDP_SESSION_TIMEOUT` | 30 | UDP 会话超时（秒） |
| `UDP_SESSION_TIMEOUT_CHECK_INTERVAL` | 10 | UDP 会话超时检查间隔（秒） |

### 通道大小

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `NETSTACK_OUTPUT_CHANNEL_SIZE` | 512 | Netstack 输出通道大小 |
| `NETSTACK_UDP_UPLINK_CHANNEL_SIZE` | 256 | Netstack UDP 上行通道大小 |
| `UDP_UPLINK_CHANNEL_SIZE` | 256 | UDP 上行通道大小 |
| `UDP_DOWNLINK_CHANNEL_SIZE` | 256 | UDP 下行通道大小 |
| `QUIC_ACCEPT_CHANNEL_SIZE` | 1024 | QUIC 接受通道大小 |
| `AMUX_ACCEPT_CHANNEL_SIZE` | 1024 | AMux 接受通道大小 |
| `AMUX_STREAM_CHANNEL_SIZE` | 16 | AMux 流通道大小 |
| `AMUX_FRAME_CHANNEL_SIZE` | 32 | AMux 帧通道大小 |

### 协议嗅探

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `TLS_DOMAIN_SNIFFING` | false | 嗅探 TLS SNI 以覆盖目标地址（仅 443 端口） |
| `TLS_DOMAIN_SNIFFING_ALL` | false | 在所有端口嗅探 TLS SNI |
| `HTTP_DOMAIN_SNIFFING` | false | 嗅探 HTTP host 以覆盖目标地址（仅 80 端口） |
| `HTTP_DOMAIN_SNIFFING_ALL` | false | 在所有端口嗅探 HTTP host |

### HTTP

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `HTTP_USER_AGENT` | （空） | HTTP 请求的 User-Agent |
| `HTTP_FORWARDED_HEADER` | X-Forwarded-For | 提取转发源 IP 的头名称 |

### 网络与 IP

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `ENABLE_IPV6` | false | 启用 IPv6 支持 |
| `PREFER_IPV6` | false | 优先使用 IPv6 |
| `OUTBOUND_INTERFACE` | 0.0.0.0,:: | 逗号分隔的绑定 IP 或接口名 |
| `OUTBOUND_DIAL_ORDER` | ordered | 拨号顺序：`ordered`, `random`, `partial-random` |

### 日志

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `LOG_CONSOLE_OUT` | false | 日志输出到控制台/标准输出 |
| `LOG_NO_COLOR` | false | 禁用日志彩色输出 |

### DNS

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `MAX_DNS_RETRIES` | 4 | DNS 查询最大重试次数 |

### 文件路径

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `ASSET_LOCATION` | （可执行文件目录） | 资源文件目录（geo.mmdb, site.dat 等） |
| `CACHE_LOCATION` | （空） | 缓存目录 |
| `API_LISTEN` | （空） | API 服务器监听地址（`addr:port`，空 = 禁用） |

### TUN 默认值

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `DEFAULT_TUN_NAME` | utun233 | 默认 TUN 接口名 |
| `DEFAULT_TUN_IPV4_ADDR` | 192.168.233.2 | 默认 TUN IPv4 地址 |
| `DEFAULT_TUN_IPV4_GW` | 192.168.233.1 | 默认 TUN IPv4 网关 |
| `DEFAULT_TUN_IPV4_MASK` | 255.255.255.0 | 默认 TUN IPv4 子网掩码 |
| `DEFAULT_TUN_IPV6_ADDR` | 2001:2::2 | 默认 TUN IPv6 地址 |
| `DEFAULT_TUN_IPV6_GW` | 2001:2::1 | 默认 TUN IPv6 网关 |
| `DEFAULT_TUN_IPV6_PREFIXLEN` | 64 | 默认 TUN IPv6 前缀长度 |

### Android

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `SOCKET_PROTECT_PATH` | （空） | Android socket 保护的 Unix socket 路径 |
| `SOCKET_PROTECT_SERVER` | （空） | Android socket 保护的 address:port |

### 特殊模式

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `GATEWAY_MODE` | false | 启用网关模式（需要 TUN） |

## Advanced Features

### TUN inbound

在 macOS 和 Linux 上还需要手动配置地址和路由表，具体可以参考 Mellow：[macOS](https://github.com/mellow-io/mellow/blob/f71f6e54768ded3cfcc46bebb706d46cb8baac08/src/main.js#L702) [Linux](https://github.com/mellow-io/mellow/blob/f71f6e54768ded3cfcc46bebb706d46cb8baac08/src/helper/linux/config_route#L1)

在 macOS 上手动配置地址：`sudo ifconfig utun7 10.10.0.2 netmask 255.255.255.0 10.10.0.1`

此外所有非组合类型的 outbound 必须正确配置一个 `bind` 地址，这是连接原网关的网卡的地址，即未连接 VPN 前网卡的 IP 地址：
```json
"outbounds": [
    {
        "bind": "192.168.0.99",
        "protocol": "shadowsocks",
        "settings": {
            "address": "x.x.x.x",
            "method": "chacha20-ietf-poly1305",
            "password": "123456",
            "port": 8389
        },
        "tag": "shadowsocks_out"
    },
    {
        "bind": "192.168.0.99",
        "protocol": "direct",
        "tag": "direct"
    }
]
```

```json
"dns": {
    "bind": "192.168.0.99",
    "servers": ["1.1.1.1"]
}
```
