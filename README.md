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

SOURCE="/tmp/tailscale.combined"
TARGET="/koolshare/bin/tailscale.combined"
NEW="${TARGET}.new.$$"

[ -f "$SOURCE" ] || { echo "源文件不存在：$SOURCE" >&2; exit 1; }
[ ! -e "$NEW" ] || { echo "临时文件已存在：$NEW" >&2; exit 1; }
[ ! -L "$TARGET" ] || { echo "拒绝替换符号链接：$TARGET" >&2; exit 1; }

trap 'rm -f "$NEW"' 0 1 2 15
if [ -f "$TARGET" ]; then
    cp -p "$TARGET" "$NEW"
    cat "$SOURCE" > "$NEW"
elif [ -e "$TARGET" ]; then
    echo "目标路径不是普通文件：$TARGET" >&2
    exit 1
else
    OWNER_GROUP="$(id -u):$(id -g)"
    cp "$SOURCE" "$NEW"
    chown "$OWNER_GROUP" "$NEW"
fi

chmod 755 "$NEW"
sync
mv -f "$NEW" "$TARGET"
trap - 0 1 2 15
sync

(cd /koolshare/bin && md5sum -c /tmp/tailscale.combined.md5)

/koolshare/scripts/tailscale_config start
/koolshare/bin/tailscale version
/koolshare/bin/tailscale status
```
