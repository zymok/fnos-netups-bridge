# 网络 UPS 桥接 · fnOS

把局域网里的 SNMP UPS 或 NUT Server 接入飞牛原生 UPS 管理。应用将 UPS 状态转换成 USB HID，在飞牛上自动挂载虚拟 CyberPower UPS。飞牛原生系统负责断电关机。

**[下载飞牛 FPK](https://github.com/zymok/fnos-netups-bridge/releases/download/v0.2.0/netups-bridge-0.2.0.fpk)** · **[预发行说明](https://github.com/zymok/fnos-netups-bridge/releases/tag/v0.2.0)** · **[修改版源码](https://github.com/zymok/fnos-netups-bridge/releases/download/v0.2.0/netups-bridge-0.2.0-source.tar.gz)**

## 上游与修改范围

本项目是 [Winmin Tam（@iwinmin）的 fnos-remote-ups](https://github.com/iwinmin/fnos-remote-ups) 的修改版。基于上游 [`bb04b5f`](https://github.com/iwinmin/fnos-remote-ups/commit/bb04b5f66a920498756b0bf414ef363d14d346c2)。感谢 Winmin Tam 编写原有的 NUT 客户端、USB/IP 服务、HID 模拟和自动挂载逻辑。

这一版新增原生 FPK、SNMP 直连、安装和运行设置向导、独立 NUT 实例及状态探测。USB/IP 服务改为只监听本机环回地址。上游 README 声明其项目使用 MIT 许可证。FPK 中打包的第三方组件各自沿用原许可证，详见 [第三方组件与源码](https://github.com/zymok/fnos-netups-bridge/releases/download/v0.2.0/THIRD_PARTY.md)。

## 安装

1. 确认飞牛是 x86_64，内核提供 USB/IP 的 `vhci-hcd`，并自带 Python 3。
2. 在 [Releases](https://github.com/zymok/fnos-netups-bridge/releases) 下载 `netups-bridge-0.2.0.fpk`。
3. 在飞牛 **应用中心 → 手动安装** 上传 FPK。按向导填写连接方式和参数。
4. 确认应用已启用，再在飞牛 **系统设置 → 硬件和电源 → UPS** 中查看虚拟 USB UPS，配置原生 UPS 保护。

FPK 自带运行所需的桥接程序、SNMP 驱动、NUT 服务和 USB/IP 工具。挂载虚拟 USB 需要 root 权限。它使用独立配置和本地端口，不修改飞牛的 `/etc/nut`。

## 配置

| 字段 | SNMP UPS | NUT Server |
| --- | --- | --- |
| 地址 | UPS 网卡的 IPv4 或主机名 | NUT Server 的 IPv4 或主机名 |
| 端口 | 默认 UDP 161 | 默认 TCP 3493 |
| SNMP 版本 | v2c 或 v1 | — |
| SNMP 只读 community | 在 UPS 管理后台配置的只读值 | — |
| SNMP MIB | 先选自动识别。兼容 RFC 1628 的设备可以试标准 UPS-MIB。 | — |
| NUT UPS 名称 | — | 填服务端定义的名字，例如 `ups`。可运行 `upsc -l <NUT主机>` 查看。 |

安装后从 **应用中心 → 已安装 → 网络 UPS 桥接 → 应用设置 → 运行设置 → 编辑** 修改。切换连接方式时清空旧端口即可使用新模式的默认值。点击「保存并重启」时，飞牛可能暂时提示 UPS 失去连接，待重新挂载后复查 UPS 状态。

NUT 模式需要服务端允许飞牛查询 UPS 状态。此版本支持 IPv4 或能解析出 IPv4 的主机名。SNMPv3、IPv6 和要求 TLS 或登录后才能查询的 NUT 服务暂未支持。

## 验证与排查

- **看不到 USB UPS**：先确认网络 UPS 或 NUT 服务可达，应用已启用。SSH 运行 `lsusb -d 0764:0501`，可以确认虚拟 USB 是否枚举。运行 `modprobe -n -v vhci-hcd` 可以检查内核模块。
- **SNMP 无数据**：检查 UDP 161、UPS 的 SNMP 版本与只读 community，以及网卡是否允许飞牛源 IP。支持标准 UPS-MIB 的设备可手选 RFC 1628。
- **NUT 无数据**：在飞牛查询 `upsc <UPS名称>@<NUT主机>:3493 ups.status`。市电正常通常返回 `OL`，电池供电通常返回 `OB`。
- **应用启动但没有虚拟 USB**：查看 `/var/apps/netups-bridge/var/bridge.log`。上游状态不可读时，桥接不会继续向飞牛提供缓存状态。
- **关机验收**：让交换机和上游 NUT 主机在断电时继续供电。在维护窗口模拟市电中断，确认飞牛能看到电池供电并按设定策略关机。若 UPS 上报的剩余续航无效，使用飞牛提供的停电计时策略，并进行实测。升级飞牛后也应复查 USB 挂载和状态。

0.2.0 已在一台 x86_64 飞牛上验证 SNMP v2c、配置保存及重启后 USB 重新挂载。NUT Server 路径已验证查询状态，更多机型、系统版本及关机联动仍待验证。
