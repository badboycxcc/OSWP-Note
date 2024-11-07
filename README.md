# OSWP-Note

## 1
更新 Kali
```
sudo apt update
sudo apt full-upgrade -y
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

恢复正常网络连接
```
sudo service networking restart
sudo service NetworkManager restart
或者
service networking start
service network-manager start
```
