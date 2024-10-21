## 任务
自建环境，抓包分析WiFi连接过程：
WiFi扫描（主动扫描；被动扫描）
WiFi认证
WiFi关联
数据传输
## 要求
使用抓包软件进行抓包分析，每一步需要截图
## 实验原理
![原理](https://github.com/user-attachments/assets/5d4f5c06-6788-4106-9324-b5f0c3e4220f)
## 实验环境
实验操作系统：Kali Linux
硬件设备：RT3070L网卡
软件：Wireshark抓包工具
## 实验过程
### 1.扫描
被动扫描：STA被动等待AP每隔一段时间定时送出的Beacon信标帧，该帧提供了AP及所在BSS相关信息
Wireshark适合进行被动扫描，并不适合主动扫描
![被动扫描](https://github.com/user-attachments/assets/5af28f7a-c16e-411a-847d-ffb21fe03b31)
在kali终端使用sudo airmon-ng start wlan0命令，将网卡设置为监听模式，以便于Wireshark进行抓包。
在Wireshark中过滤器中使用wlan.fc.type_subtype == 0x8筛选，把所有Beacon帧过滤出来
![过滤出Beacon帧](https://github.com/user-attachments/assets/7fdc6306-09a7-410c-9e86-f3ba2c9df890)
点开其中一个信标帧，即可查看AP及所在BSS相关信息
![查看一个Beacon帧](https://github.com/user-attachments/assets/90851c06-879b-43c7-93e9-8d353b4e520b)
### 2.认证
为了保证无线链路安全，接入过程中AP需要完成对客户端的认证，只有通过认证后才能进入后续的关联阶段。802.11链路协议定义了两种认证机制：开放系统认证和共享密钥认证。
开放系统认证：不认证、不加密，只要WLAN服务段支持该认证方式，WLAN客户端就可以链路认证成功
共享密钥认证：客户端和服务段配置相同的共享密钥，WLAN服务端在链路认证过程验证两边的密钥配置是否相同，如果一致，则认证成功，否则认证失败。
这里我们采用的是开放系统认证机制
![开放系统认证](https://github.com/user-attachments/assets/b50b5904-4da6-46b8-a1fd-69f21229469d)
在Wireshark中，我们在过滤器输入wlan.fc.type_subtype == 0xB即可将Authentication数据帧过滤出
![过滤Authentication](https://github.com/user-attachments/assets/6729240f-8a2b-4268-a929-8a39bb338fe4)
通过查看Authentication数据帧的信息，我们可以看到，已经认证成功了
![查看Authentication](https://github.com/user-attachments/assets/231c5c86-9a3e-472c-8c52-57f34f6c6c5b)
### 3.关联
![关联](https://github.com/user-attachments/assets/ed325401-f660-4d69-869a-3879c80853ee)
在Wireshark中，我们在过滤器输入wlan.fc.type_subtype == 0x0 || wlan.fc.type_subtype == 0x1即可将Association Request 和Association Response过滤出来
![过滤response](https://github.com/user-attachments/assets/397320cd-f0b3-4d77-9d05-e59a45dfb492)
如图所示，通过前两条数据帧，我们可以看到，WLAN客户端和WLAN服务端成功完成链路服务协商，表明两个设备成功建立了802.11链路