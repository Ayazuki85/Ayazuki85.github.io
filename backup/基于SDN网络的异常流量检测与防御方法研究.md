## 完成本课题需要做的工作包括
1. 理解软件定义网络（Software-Defined Networking，SDN）的基本概念和原理。
2. 设计并搭建一个SDN网络实验环境，包括控制器、交换机和主机等设备。
3. 了解SDN网络中存在的攻击类型，例如DDOS攻击；学习并掌握DDOS攻击原理。
4. 理解sFlow-rt的基本配置与操作；
5. 掌握通过floodlight进行DDoS防御的原理；
6. 掌握mininet中sFlow agent的配置过程；
## 实验环境
一台Ubuntu22.04虚拟机，配置有mininet、ryu控制器、Apifox、sFlow-rt 3.0等工具。
## 实验过程
1.启动mininet端，首先构造一个拓扑，拓扑的python代码如下
```python
from mininet.topo import Topo
from mininet.net import Mininet
from mininet.node import RemoteController

class MyTopo(Topo):
    def build(self):
        s1 = self.addSwitch('s1')
        h1 = self.addHost('h1', ip='10.0.0.1/24')
        h2 = self.addHost('h2', ip='10.0.0.2/24')
        h3 = self.addHost('h3', ip='10.0.0.3/24')
        self.addLink(h1, s1)
        self.addLink(h2, s1)
        self.addLink(h3, s1)

if __name__ == '__main__':
    topo = MyTopo()
    net = Mininet(topo=topo, controller=RemoteController)
    net.start()
    net.pingAll()
    net.interact()
    net.stop()
```
该代码构成的拓扑图如下
![可视化拓扑图](https://github.com/user-attachments/assets/50d17abd-6873-49d3-b51f-1c511ee96f78)
2.以指定命令启动mininet并运行该拓扑
```
 sudo mn --controller=remote,ip=127.0.0.1,port=6653 --switch=ovsk --topo=single,3 --mac --custom my_topo.py
```
这里需要注意应该指定mininet启动的端口，如果mininet启用默认端口，后续则会和Ryu控制器默认端口冲突导致Ryu控制器启动失败。
3.启动Ryu控制器接管该拓扑中的控制器
```
ryu-manager --observe-links ryu/ryu/app/gui_topology/gui_topology.py ryu/ryu/app/simple_switch_13.py
```
![RYU启动](https://github.com/user-attachments/assets/0a0d828c-50fa-4d66-89e3-7186701c9f4f)
4.配置sFlow Agent
我们需要在虚拟交换机配置sFlow Agent，这样sFlow Collector 才能收集到流量信息进行分析和呈现。键入以下指令部署sFlow Agent。配置命令如下：
```
sudo ovs-vsctl -- --id=@sflow create sflow agent=s1-eth0 target=\"127.0.0.1:6343\" sampling=10 polling=20 -- -- set bridge s1 sflow=@sflow

./start.sh
```
![配置Agent](https://github.com/user-attachments/assets/1e696b75-b82c-4723-8a3d-12b625de2a83)
5.在浏览器中输入网址localhost:8008/html/index.html，我们可以通过其WebUI进行查看Agent 是否配置成功，配置成功的话可以看见各项监控状态。
![是否配置成功](https://github.com/user-attachments/assets/6989147c-3404-4288-b1e4-a962eade870a)
6.我们现在切换到mininet终端
使用如下指令：```xterm h1 h2 ```，打开 Host1，和Host2的终端；
然后在 Host1 上启动一个 http 服务：```python3 -m http.server 80&```
此时让h2正常的 ping h1，观察此时的情况，未发现异常。
![正常Ping](https://github.com/user-attachments/assets/978e76d2-a2c2-4c16-92d7-f38786719fde)
7.现在开始模拟DDoS攻击
在h2终端上使用命令``` h2 ping -f 10.0.0.1```
再观察此时的情况
![被DDOS](https://github.com/user-attachments/assets/5ffbf5de-4fe4-4da6-b17a-42e0f2120de7)
8.现在进行DDOS防御
我们在监测到流量异常之后，需要利用RYU控制器向OpenFlow交换机下发流表，抑制攻击流量。由于前面的DDoS采用的ping洪范攻击，因此我们需要下发流表，将攻击流down掉，因此利用Apifox添加流表。具体操作：首先进入Apifox，利用get操作可以获得已有的流表，所以url填http://localhost:8080/stats/flow/1
![防御1](https://github.com/user-attachments/assets/9ee7d51c-961b-495b-9632-5828ca9f2398)
为了抑制攻击流，我们需要添加流表。
我们下发流表控制从交换机的端口1经过的流量，url填写为 http://localhost:8080/stats/flowentry/add
功能选择post方式，在Body中选择JSON进行流表的编写。
dpid代表交换机的号
priority表示优先级，优先级数字越大表示越优先处理
在match写入的in_port代表交换机输入端口为1的端口
dl_type字段代表ip协议，其中2048的16进制0x0800代表ipv4协议
nw_proto代表:ip协议上搭载的协议类型，其中1代表icmp协议
在本实验中也就是h1连接交换机的端口，actions中为空，表示交换机对于h1的流量不作任何处理。编写完成后，点击send进行流表下发，如果发送成功会返回200 OK。
![防御2](https://github.com/user-attachments/assets/ae6cf747-738b-4bb9-9cd9-416b118ae1f9)
此时我们再观察一下情况，发现情况已经趋向于正常
![正常](https://github.com/user-attachments/assets/29c54bec-49a3-4406-81e8-aa194d42f0bd)
# 参考文献:
Ubuntu22.04安装mininet和ryu：
https://blog.csdn.net/2301_76203161/article/details/135979942
SDN实验：使用mininet和RYU实现DDoS攻击与防御模拟: 
https://blog.csdn.net/weixin_45236003/article/details/132622991
sFlow-rt 3.0流量监控工具安装部署及简单实验：
https://blog.csdn.net/Call_Me_JHZ/article/details/105465768
Mininet+Ryu安装教程_ubuntu安装mininet和ryu最新csdn
https://blog.csdn.net/2401_84010784/article/details/137664161