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

OWNER_GROUP="$(stat -c '%u:%g' /koolshare/bin/tailscale.combined)"
cp -f /tmp/tailscale.combined /koolshare/bin/tailscale.combined
chmod 755 /koolshare/bin/tailscale.combined
chown "$OWNER_GROUP" /koolshare/bin/tailscale.combined

/koolshare/scripts/tailscale_config start
/koolshare/bin/tailscale version
/koolshare/bin/tailscale status
```
