---
description: Check USB devices, ADB, and fastboot status for Android phone flashing
---

Diagnose USB-connected Android devices — check connectivity, device info, and readiness for flashing.

## Steps

1. **List all USB devices**
   ```bash
   lsusb
   ```
   Identify the Android device by vendor ID (Xiaomi: `2717`, Samsung: `04e8`, Google: `18d1`).

2. **Check ADB connection**
   ```bash
   adb devices
   ```
   Possible states:
   - `device` — ready to use
   - `unauthorized` — user must tap "Allow USB debugging" on phone screen
   - `offline` — reconnect USB cable
   - (empty) — no device detected, check cable/driver

3. **Check Fastboot mode**
   ```bash
   fastboot devices 2>/dev/null || echo "fastboot not available or no devices"
   ```
   If no fastboot device, guide user to enter fastboot:
   - Power off phone
   - Hold **Power + Volume Down** until fastboot screen appears

4. **Get device model and specs** (only when ADB is authorized)
   ```bash
   adb shell getprop ro.product.model
   adb shell getprop ro.product.brand
   adb shell getprop ro.product.device
   adb shell getprop ro.build.version.release
   adb shell getprop ro.product.board
   ```

5. **Check storage partitions** (useful for flashing context)
   ```bash
   adb shell "cat /proc/partitions" 2>&1 | grep -E "mmcblk|sda"
   ```

## Common issues

- **"unauthorized"**: User needs to confirm USB debugging popup on phone
- **No fastboot**: Phone must be in bootloader mode, not normal Android mode
- **Multiple devices**: Use `adb -s <serial>` to target a specific device

## Stopping condition

Report device info and connection status. If flashing is the goal, confirm fastboot is reachable and guide the next step.
