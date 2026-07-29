# Tailscale Router IPQ64 Build

## 替换

将 Release 中的 `tailscale.combined` 和 `tailscale.combined.md5` 下载到路由器的 `/tmp`，然后执行：

```sh
set -e
cd /tmp
md5sum -c tailscale.combined.md5

/koolshare/scripts/tailscale_config stop
killall tailscale 2>/dev/null || true
killall tailscaled 2>/dev/null || true

BAK="/koolshare/tailscale_bak_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$BAK"
cp -a /koolshare/bin/tailscale* "$BAK"/

TARGET="/koolshare/bin/tailscale.combined"
[ -f "$TARGET" ] || { echo "目标文件不存在：$TARGET" >&2; exit 1; }
cat /tmp/tailscale.combined > "$TARGET"
chmod 755 "$TARGET"

/koolshare/scripts/tailscale_config start
/koolshare/bin/tailscale version
/koolshare/bin/tailscale status
```
