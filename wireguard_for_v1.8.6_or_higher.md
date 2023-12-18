### 使用 **warp-reg**，注册warp账号

```
curl -sLo warp-reg https://github.com/badafans/warp-reg/releases/download/v1.0/main-linux-amd64 && chmod +x warp-reg && ./warp-reg && rm warp-reg
```

### 使用 **api.zeroteam.top**，获取warp账号

```
curl -sL "https://api.zeroteam.top/warp?format=sing-box" | grep -Eo --color=never '"2606:4700:[0-9a-f:]+/128"|"private_key":"[0-9a-zA-Z\/+]+="|"reserved":\[[0-9]+(,[0-9]+){2}\]'
```

- 复制输出的 IPv6 地址，替换下面配置中的 `2606:4700::`
- 复制输出的 `private_key/secretKey` 值，粘贴到下面配置中 `secretKey` 后的 `""` 中
- 复制输出的 `reserved` 值，粘贴到下面配置中 `reserved` 后的 `[]` 中

### "outbounds"
```jsonc
        {
            "protocol": "wireguard",
            "settings": {
                "secretKey": "", // 粘贴你的 "private_key" 值
                "address": [
                    "172.16.0.2/32",
                    "2606:4700::/128" // 粘贴你获得的 warp IPv6 地址，结尾加 /128
                ],
                "peers": [
                    {
                        "publicKey": "bmXOC+F1FxEMF9dyiK2H5/1SUtzH0JuVo51h2wPfgyo=",
                        "allowedIPs": [
                            "0.0.0.0/0",
                            "::/0"
                        ],
                        "endpoint": "162.159.192.1:2408" // 或填写 engage.cloudflareclient.com:2408
                    }
                ],
                "reserved":[0, 0, 0], // 粘贴你的 "reserved" 值
                "mtu": 1280,
                "domainStrategy": "ForceIPv6v4" // 若需使用Cloudflare的IPv4，改为 "ForceIPv4"
            },
            "tag": "wireguard"
        }
```

| domainStrategy | [test-ipv6.com](https://test-ipv6.com/) | [bgp.he.net](https://bgp.he.net/) | [chat.openai.com](https://chat.openai.com/cdn-cgi/trace)
| :--- | :---: | :---: | :---: |
| ForceIPv6v4 | IPv6v4地址 | IPv6地址 | IPv6地址 |
| ForceIPv6 | 网站打不开 | IPv6地址 | IPv6地址 |
| ForceIPv4v6 | IPv6v4地址 **1** | IPv4地址 | IPv4地址 |
| ForceIPv4 | IPv4地址 | IPv4地址 | IPv4地址 |
| ForceIP | IPv6v4地址 **2** | IPv6地址 | IPv6地址 |

**1：** 提示`你已经有 IPv6 地址了，但你的浏览器不太愿意用，这一点比较令人担心。`<br>
**2：** 有机率提示`你已经有 IPv6 地址了，但你的浏览器不太愿意用，这一点比较令人担心。`

编辑 **/usr/local/etc/xray/config.json**，按需增加 **"routing"**，**"inbounds"**，**"outbounds"** 的内容（注意检查json格式），输入 `systemctl restart xray` 重启Xray，访问[chat.openai.com/cdn-cgi/trace](https://chat.openai.com/cdn-cgi/trace)查看是否为Cloudflare的IP。

### "routing"
```jsonc
            {
                "type": "field",
                "domain": [
                    "geosite:openai"
                ],
                "outboundTag": "wireguard"
            }
```

### "inbounds"
```jsonc
            "sniffing": {
                "enabled": true,
                "destOverride": [
                    "http",
                    "tls",
                    "quic"
                ]
            }
```

### 服务端配置示例

```jsonc
{
    "log": {
        "loglevel": "warning"
    },
    "routing": {
        "domainStrategy": "IPIfNonMatch",
        "rules": [
            {
                "type": "field",
                "domain": [
                    "geosite:openai"
                ],
                "outboundTag": "wireguard"
            },
            {
                "type": "field",
                "ip": [
                    "geoip:cn"
                ],
                "outboundTag": "wireguard"
            }
        ]
    },
    "inbounds": [
        {
            // 粘贴你的服务端配置
            "sniffing": {
                "enabled": true,
                "destOverride": [
                    "http",
                    "tls",
                    "quic"
                ]
            }
        }
    ],
    "outbounds": [
        {
            "protocol": "freedom",
            "tag": "direct"
        },
        {
            "protocol": "blackhole",
            "tag": "block"
        },
        {
            "protocol": "wireguard",
            "settings": {
                "secretKey": "", // 粘贴你的 "private_key" 值
                "address": [
                    "172.16.0.2/32",
                    "2606:4700::/128" // 粘贴你获得的 warp IPv6 地址，结尾加 /128
                ],
                "peers": [
                    {
                        "publicKey": "bmXOC+F1FxEMF9dyiK2H5/1SUtzH0JuVo51h2wPfgyo=",
                        "allowedIPs": [
                            "0.0.0.0/0",
                            "::/0"
                        ],
                        "endpoint": "162.159.192.1:2408" // 或填写 engage.cloudflareclient.com:2408
                    }
                ],
                "reserved":[0, 0, 0], // 粘贴你的 "reserved" 值
                "mtu": 1280,
                "domainStrategy": "ForceIPv6v4" // 若需使用Cloudflare的IPv4，改为 "ForceIPv4"
            },
            "tag": "wireguard"
        }
    ]
}
```