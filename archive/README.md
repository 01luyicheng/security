# Archive

本目录存放历史扫描结果和临时文件。

## 目录结构

```
archive/
├── scan-results/          # 网络扫描原始结果
│   ├── nmap_fullscan.txt  # Nmap全端口扫描 (1.5MB)
│   ├── nmap_discovery.gnmap  # 主机发现结果
│   ├── nmap_ports_13.txt  # 13号端口扫描
│   ├── nmap_ports_73.txt  # 73号端口扫描
│   ├── nmap_services.txt  # 服务识别结果
│   └── dirb_80.txt        # 目录爆破结果
└── camera-captures/       # 摄像头抓拍
    └── 192.168.20.45_snapshot.jpg
```

## 说明

- 扫描结果来自2026年6月的网络审计
- 原始数据保留用于参考，综合报告见 `reports/` 目录
