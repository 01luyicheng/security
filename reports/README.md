# Security Reports

This repository contains security audit and penetration testing reports.

## Reports

### 1. [192.168.20.0/24 Audit Report](192.168.20.0_24_audit_report.md)
- **Date**: 2026-06-18
- **Scope**: 192.168.20.0/24 subnet
- **Type**: Network security audit
- **Key Findings**: 
  - Unauthorized Xiaomi router (192.168.20.23)
  - Windows 7 SP1 vulnerable host (192.168.20.19)
  - RDP exposure (192.168.20.14, 192.168.20.50)
  - Hikvision cameras with RTSP open

### 2. [九峰职业学校 Network Penetration Test](九峰职业学校网络渗透测试报告.md)
- **Date**: 2026-01-08 to 2026-06-11
- **Scope**: School network (192.168.20.0/24, 10.5.190.0/24)
- **Type**: Penetration test
- **Key Findings**:
  - Critical: MQTT Broker unauthorized access (CVSS 9.8)
  - High: Xiaomi router authentication bypass
  - Network topology fully exposed via MQTT
  - 188 online users identified

### 3. [Mihomo Configuration Optimization](mihomo优化规划.md)
- **Type**: Configuration optimization plan
- **Key Changes**:
  - Disable IPv6 (current network not ready)
  - Enable tcp-concurrent
  - Create policy groups (AI, STREAM, PROXY)
  - Security hardening (secret, allow-lan)

## Directory Structure

```
security/
├── reports/
│   ├── 192.168.20.0_24_audit_report.md
│   ├── 九峰职业学校网络渗透测试报告.md
│   └── mihomo优化规划.md
├── nmap_scans/
├── camera_captures/
└── ...
```

## Usage

```bash
# View audit report
cat reports/192.168.20.0_24_audit_report.md

# View penetration test report
cat reports/九峰职业学校网络渗透测试报告.md
```
