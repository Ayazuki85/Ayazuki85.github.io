## 背景

公司的主机安装的Linux系统是debian13 + GNOME，由于Wayland协议成为了最近的Linux发行版的主流，目前国产的todesk，向日葵都不支持Wayland，由于Linux用户基数少，国产软件适配是指望不上了。急需一份Wayland下的远程控制方案来解燃眉之急，经过数日搜索和尝试，整理出以下两种方案。为了实现远程控制我购置了一台云服务器，规格为 公网 IP，2 核 2G，3M 带宽。我的需求是在家里使用自己的Windows笔记本电脑远程连接公司的Debian 13主机。

## 一、Rustdesk + 自己搭建的中继服务器

### 1.中继服务器端配置

#### 1.1 安装docker

```bash
# 首先要配置好docker
sudo apt install docker.io docker-compose
# 将docker加入用户组
sudo usermod -aG docker $USERnewgrp docker
# 检查docker是否启动
sudo systemctl status docker
# 如果没有启动就启动docker
sudo systemctl start dockersudo systemctl enable docker
```

#### 1.2 配置Rustdesk中继服务器的docker容器

创建Rustdesk目录用来放置相关内容

`mkdir rustdesk`
`cd ruskdesk/`

创建 `docker-compose.yml`

`vim docker-compose.yml`

输入以下内容

```yml
version: '3'
services:
  hbbs:
    image: rustdesk/rustdesk-server:latest
    container_name: rustdesk-hbbs
    command: hbbs -r XXX.XXX.XXX.XXX:21117  # 这里是你的公网IP，按需修改,也可以填写域名替代。
    volumes:
      - ./data:/root
    network_mode: host
    restart: unless-stopped
  hbbr:
    image: rustdesk/rustdesk-server:latest
    container_name: rustdesk-hbbr
    command: hbbr
    volumes:
      - ./data:/root
    network_mode: host
    restart: unless-stopped
```

**根据 `docker-compose.yml` 配置文件，**在后台启动并运行容器应用

`docker-compose up -d`

保存下面的这个密钥，后续客户端需要

`cat ./data/id_ed25519.pub`

#### 1.3 配置服务器外面的网络防火墙。

需要开放以下端口

| 端口    | 协议  | 用途     |
| ----- | --- | ------ |
| 21115 | TCP | NAT 测试 |
| 21116 | TCP | 信令服务   |
| 21116 | UDP | 心跳服务   |
| 21117 | TCP | 中继服务   |



#### 1.4 测试

- 在服务器上监听端口
  
   `ss -tulnp | grep -E '21115|21116|21117'`

- 在本地测试端口
  
   `Test-NetConnection -ComputerName xxx.xxx.xxx.xxx -Port 21115`
  
   显示`TcpTestSucceeded : True`说明服务器端以及配置完成

### 2.Debian 13 被控端配置

#### 2.1 安装Rustdesk

找到合适的Rustdesk的安装包，使用`sudo apt install ./安装包名` 安装即可

#### 2.2 Rustdesk 配置

打开 RustDesk，系统会自动请求屏幕采集权限，点击允许

在rustdesk的客户端找到设置-网络-ID中继服务器

| 配置项   | 内容                    |
| ----- |:---------------------:|
| ID服务器 | xxx.xxx.xxx.xxx:21116 |
| 中继服务器 | xxx.xxx.xxx.xxx:21117 |
| key   | 之前保存的密钥               |

在这里可以将配置好的设置导出，方便后续在主控端配置

接下来在设置 → 安全 → 设置永久密码，并记录下被控端的ID和密码

### 3.Windows 主控端配置

#### 3.1 安装Rustdesk

找到合适的Rustdesk安装包，安装。

#### 3.2 Rustdesk 配置

在rustdesk的客户端找到设置-网络-ID中继服务器，将之前保留的配置导入即可

#### 3.3 测试连接

输入被控端的ID和密码，这里需要注意如果是第一次连接需要在被控端授权才能远程控制。连接成功后可以在历史记录中找到被控端的设置选择`强制走中继连接`

**注意**：我这里遇到了一个问题，我在过了一段时间后使用Rustdesk去远程控制服务器，发现仍需要被控端进行一次授权，否则无法正常远程控制，基于Rustdesk进行远程控制还是不够稳定，针对此问题，我推荐使用第二种方案。

## 二、内网穿透+Windows自带的远程桌面连接(推荐)

### 1. Debian 13 被控端配置

#### 1.1 设置好远程桌面

由于我这边的 Debian 13 使用 Wayland + GNOME，选择 `gnome-remote-desktop`（原生支持 Wayland）。

```bash
sudo apt update
sudo apt install gnome-remote-desktop
sudo systemctl enable --now gnome-remote-desktop
```

然后进入 **设置 → 共享 → 远程桌面**：

- 开启「远程桌面」。
- 设置用户名、密码。
- 确认监听 `127.0.0.1:3389`（默认）。

验证：

```bash
sudo ss -tlnp | grep 3389
```

#### 1.2 部署 frp

从 frp release 下载 `linux_amd64` 包，解压到 `/usr/local/frp/`：

```bash
sudo mkdir -p /usr/local/frp
cd /usr/local/frp
sudo wget https://github.com/fatedier/frp/releases/download/v0.XX.0/frp_0.XX.0_linux_amd64.tar.gz
sudo tar -xzf frp_0.XX.0_linux_amd64.tar.gz --strip-components=1
```

#### 1.3 配置文件 `/usr/local/frp/frpc.toml`

```toml
serverAddr = "你的云服务器公网IP"
serverPort = 7000
auth.method = "token"
auth.token = "你的强token"
[[proxies]]
name = "debian-rdp-1"
type = "tcp"
localIP = "127.0.0.1"
localPort = 3389
remotePort = 13389
[[proxies]]
name = "debian-rdp-2"
type = "tcp"
localIP = "127.0.0.1"
localPort = 3390
remotePort = 13390
[[proxies]]
name = "debian-ssh"
type = "tcp"
localIP = "127.0.0.1"
localPort = 22
remotePort = 2222
```

#### 1.4 配置systemd 服务 `/etc/systemd/system/frpc.service`

```ini
[Unit]
Description=frp client
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/frp/frpc -c /usr/local/frp/frpc.toml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

启用并启动：

```bash
sudo chmod +x /usr/local/frp/frpc
sudo systemctl daemon-reload
sudo systemctl enable --now frpc
sudo systemctl status frpc
```

### 2. 在云服务器端部署 frp

#### 2.1 部署frp

从 frp release 下载 `linux_amd64` 包，解压到 `/usr/local/frp/`：

```bash
sudo mkdir -p /usr/local/frp
cd /usr/local/frp
sudo wget https://github.com/fatedier/frp/releases/download/v0.XX.0/frp_0.XX.0_linux_amd64.tar.gz
sudo tar -xzf frp_0.XX.0_linux_amd64.tar.gz --strip-components=1
```

#### 2.2 配置文件 `/usr/local/frp/frps.toml`

```toml
bindPort = 7000
auth.method = "token"
auth.token = "你的强token"
allowPorts = [
  { single = 13389 },
  { single = 13390 },
  { single = 2222 },
]
# 限制 Debian 端只能申请这三个端口
```

#### 2.3 systemd 服务 `/etc/systemd/system/frps.service`

```ini
[Unit]
Description=frp server
After=network.target
[Service]
Type=simple
ExecStart=/usr/local/frp/frps -c /usr/local/frp/frps.toml
Restart=on-failure
RestartSec=5
[Install]
WantedBy=multi-user.target
```

启用并启动：

```bash
sudo chmod +x /usr/local/frp/frps
sudo systemctl daemon-reload
sudo systemctl enable --now frps
sudo systemctl status frps
```

#### 2.4 防火墙 / 安全组规则

| 方向  | 端口    | 来源        | 说明        |
| --- | ----- | --------- | --------- |
| 入站  | 7000  | 0.0.0.0/0 | frpc 控制连接 |
| 入站  | 13389 | 0.0.0.0/0 | RDP 入口 1  |
| 入站  | 13390 | 0.0.0.0/0 | RDP 入口 2  |
| 入站  | 2222  | 0.0.0.0/0 | SSH 暴露端口  |

### 3. Windows端连接方式

无需安装任何额外软件即可远程连接。

#### 3.1 RDP 远程桌面

1. 按 `Win + R`，输入 `mstsc`，回车(也可以 `Win + S`，输入远程桌面连接，回车)。

2. 点击左下角的`显示选项`，在计算机栏填写 `你的云服务器公网IP:13390`。在用户名栏填写设置的好被控端用户名，我这里是`user` (需要注意，在这填写服务器绑定的域名则会连接报错)

3. 在选项卡`显示`中将显示配置拉成全屏，将颜色设置为`增强色(16位)`

4. 在选项卡`体验`中选择低速宽带(256kbps - 2 Mbps)，并取消勾选 桌面背景、字体平滑、桌面合成、拖动时显示窗口内容、菜单和窗口动画 和 视觉样式；可以勾选持久位图缓存和如果连接中断则重新连接。

5. 在选项卡`高级`中的服务器身份验证中设置如果服务器身份验证失败则`连接并且不显示警告`。

6. 输入 Debian 上 `gnome-remote-desktop` 设置的密码。

#### 3.2 SSH 连接

打开 PowerShell 或命令提示符：

```powershell
ssh 被控端用户名@云服务器公网IP(或者是域名) -p 2222
# 这里需要注意的是一定要指定端口号为2222
# 不带端口号的话是ssh连接的是中继服务器。
```

首次连接会提示确认主机指纹，输入 `yes`，然后输入服务器的锁屏密码。
