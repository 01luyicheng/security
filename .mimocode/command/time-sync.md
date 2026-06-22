---
description: Diagnose and fix system time sync issues on Linux
---

Diagnose and fix system time synchronization on this Linux machine.

## Steps

1. **Check current time and timezone**
   ```bash
   date && timedatectl 2>/dev/null || echo "timedatectl not available"
   ```
   Look for "System clock synchronized: no" — this is the root problem when present.

2. **Check which NTP service is running**
   ```bash
   systemctl status systemd-timesyncd 2>/dev/null | head -20
   systemctl status chronyd 2>/dev/null | head -10
   systemctl status ntp 2>/dev/null | head -10
   ```

3. **Test NTP server reachability** (try multiple Chinese/international servers)
   ```bash
   ping -c 2 -W 3 ntp.aliyun.com 2>&1
   ping -c 2 -W 3 ntp.cloud.tencent.com 2>&1
   ping -c 2 -W 3 time.google.com 2>&1
   ```
   If ICMP is blocked, test HTTP to confirm internet works:
   ```bash
   curl -s --max-time 5 http://www.baidu.com -o /dev/null -w "%{http_code}"
   ```

4. **Attempt manual time sync**
   ```bash
   # Try ntpdate if available
   if command -v ntpdate &>/dev/null; then
       sudo ntpdate -u ntp.aliyun.com 2>&1
   # Try chronyc
   elif command -v chronyc &>/dev/null; then
       sudo chronyc makestep 2>&1
   # Try systemd-timesyncd
   else
       sudo timedatectl set-ntp true 2>&1
       sudo systemctl restart systemd-timesyncd 2>&1
   fi
   ```

5. **Verify sync succeeded**
   ```bash
   date && timedatectl 2>/dev/null | grep -i "synchronized"
   ```

## Common issues

- **NTP UDP 123 blocked by firewall**: Try alternative servers or use HTTPS-based time API as workaround
- **systemd-timesyncd idle but not synced**: Restart the service, or switch to chrony
- **RTC hardware clock drifted**: `sudo hwclock --systohc` to write system time to RTC

## Stopping condition

Report success when "System clock synchronized: yes" or when the root cause is clearly identified and communicated to the user.
