# Tailscale Router IPQ64 Build

用于 koolshare / 华硕 IPQ64 路由器的 Tailscale `linux/arm64` 自定义构建。

本仓库通过 GitHub Actions 从 Tailscale 官方源码构建 `tailscale.combined`，用于替换 koolshare Tailscale 插件中的 combined 二进制文件。

> 非官方构建。Tailscale 项目由 Tailscale Inc. 开发维护，本仓库仅提供特定路由器环境下的自动构建流程。

## 适用环境

```text
架构：aarch64 / arm64
平台：IPQ64
插件：koolshare Tailscale
模式：tailscale / tailscaled -> tailscale.combined
```

典型文件结构：

```text
/koolshare/bin/tailscale -> tailscale.combined
/koolshare/bin/tailscaled -> tailscale.combined
/koolshare/bin/tailscale.combined
```

## 构建说明

构建目标：

```text
GOOS=linux
GOARCH=arm64
GOARM64=v8.0
CGO_ENABLED=0
```

主要特性：

```text
combined tailscale + tailscaled
UPX --lzma --best
保留 koolshare 插件常用功能
加入 ts_omit_gro
```

使用的 build tags：

```text
ts_omit_aws,ts_omit_bird,ts_omit_tap,ts_omit_kube,ts_omit_completion,ts_include_cli,ts_omit_systray,ts_omit_tpm,ts_omit_debugeventbus,ts_omit_syspolicy,ts_omit_capture,ts_omit_ssh,ts_omit_taildrop,ts_omit_gro
```

其中 `ts_omit_gro` 用于规避部分 IPQ5332 / Linux 5.4 环境下可能出现的 `invertGSOChecksum` panic。

## 版本策略

自动构建以 Tailscale 官方 stable 页面中的 **Linux arm64 包版本** 为准：

```text
https://pkgs.tailscale.com/stable/
```

不会单纯追随 GitHub 最新 tag，以避免构建仅适用于其他平台的版本。

## 自动构建

工作流每天定时检查一次版本。

如果对应 Release 已存在，则跳过构建；如果发现新的 Linux arm64 stable 版本，则自动构建并发布。

也可以在 Actions 页面手动运行：

```text
Actions -> Build Tailscale Router IPQ64 Binary -> Run workflow
```

## Release 文件

```text
tailscale.combined          路由器使用的 UPX 压缩版
tailscale.combined.md5      路由器端 MD5 校验文件
buildinfo.txt               Go 构建信息
upx-info.txt                UPX 压缩信息
```

GitHub Release 页面自动附带的 `Source code (zip)` 和 `Source code (tar.gz)` 不是构建产物，一般无需下载。

## 路由器端校验

```sh
cd /tmp
md5sum -c tailscale.combined.md5
```

正常输出：

```text
tailscale.combined: OK
```

## 手动替换

停止插件：

```sh
/koolshare/scripts/tailscale_config stop
killall tailscale 2>/dev/null
killall tailscaled 2>/dev/null
sleep 2
killall -9 tailscale 2>/dev/null
killall -9 tailscaled 2>/dev/null
```

备份旧文件：

```sh
BAK=/koolshare/bin/tailscale_bak_$(date +%Y%m%d_%H%M%S)
mkdir -p "$BAK"
cp -a /koolshare/bin/tailscale* "$BAK"/
```

替换二进制：

```sh
cp -f /tmp/tailscale.combined /koolshare/bin/tailscale.combined
chmod 755 /koolshare/bin/tailscale.combined
chown sun:root /koolshare/bin/tailscale.combined 2>/dev/null
```

启动并检查：

```sh
/koolshare/bin/tailscale version
/koolshare/scripts/tailscale_config start
/koolshare/bin/tailscale status
```

## 回滚

```sh
/koolshare/scripts/tailscale_config stop
cp -af "$BAK"/tailscale* /koolshare/bin/
chmod 755 /koolshare/bin/tailscale.combined 2>/dev/null
/koolshare/scripts/tailscale_config start
/koolshare/bin/tailscale version
```

## 免责声明

本仓库不是 Tailscale 官方发布渠道。请在了解风险并保留旧版本备份后使用。
