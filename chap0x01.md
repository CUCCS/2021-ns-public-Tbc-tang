# NS chap0x01 基于 VirtualBox 的网络攻防基础环境搭建

## 实验目的

- 掌握 VirtualBox 虚拟机的安装与使用；
- 掌握 VirtualBox 的虚拟网络类型和按需配置；
- 掌握 VirtualBox 的虚拟硬盘多重加载；

## 实验环境

以下是本次实验需要使用的网络节点说明和主要软件举例：

- VirtualBox 虚拟机

- 攻击者主机（Attacker）：Kali Rolling 2109.2

- 网关（Gateway, GW）：Debian Buster

- 靶机（Victim）：From Sqli to shell / xp-sp3 / Kali

- 相关[下载链接](https://c4pr1c3.gitee.io/cuc-wiki/ns/2021/calendar.html)，请使用cuc校园网下载

 > 自己去**官网**下载配置也行，只是略显麻烦，这里就偷个懒

## 实验要求

- 虚拟硬盘配置成多重加载，效果如下图所示；

<img src="https://c4pr1c3.github.io/cuc-ns/chap0x01/attach/chap0x01/media/vb-multi-attach.png" alt="img" style="zoom:60%;" />

- 搭建满足如下拓扑图所示的虚拟机网络拓扑；

<img src="https://c4pr1c3.github.io/cuc-ns/chap0x01/attach/chap0x01/media/vb-exp-layout.png" alt="img" style="zoom:50%;" />

> 根据实验宿主机的性能条件，可以适度精简靶机数量

- 完成以下网络连通性测试；
  - [x] 靶机可以直接访问攻击者主机
  - [x] 攻击者主机无法直接访问靶机
  - [x] 网关可以直接访问攻击者主机和靶机
  - [x] 靶机的所有对外上下行流量必须经过网关
  - [x] 所有节点均可以访问互联网

---

## 实验步骤

### 虚拟硬盘配置成多重加载

- 打开“管理”-“虚拟介质管理” 或 直接CTRL+D
- 选中对应虚拟盘，释放盘片并将其改为多重加载

<img src="/img/多重加载.png" style="zoom:39%;" />

---

### 搭建网络拓扑图

- 本次实验需要用到六台虚拟机，分别为两台XP系统，两台Debian系统以及两台Kali系统，一台Kali作为攻击者，一台Debian作为网关，剩下四台分别在两个局域网内，均作为靶机。

- 简单总结一下VirtualBox中的网络类型

| 网络类型  | 虚拟机对主机的访问 | 主机对虚拟机的访问 | 虚拟机对其他主机的访问 | 其他主机对虚拟机的访问 | 虚拟机之间       |
| --------- | ------------------ | ------------------ | ---------------------- | ---------------------- | ---------------- |
| NAT网络   | √                  | ×                  | √                      | ×                      | ×                |
| 网桥      | √                  | √                  | √                      | √                      | √                |
| 内网      | ×                  | ×                  | ×                      | ×                      | 仅同一网络内可以 |
| Host-only | ×                  | ×                  | ×                      | ×                      | √                |



* 配置拓扑环境需要的各网络

  * 网关所需网卡：NAT网络（使网关可以访问攻击者主机），Host-only网络（方便我们进行ssh连接），两块内部网络intnet1/2（分别用于搭建两块独立的局域网）

  <img src="/img/network_gateway.png" style="zoom:39%;" />

  ​	登录网关虚拟机器以查看其IP地址，我建议先为网关虚拟机[创建一个新的有root权限的用户](https://www.jianshu.com/p/a60340df9004?utm_campaign=)，这样就可以顺便[配置一下ssh](https://blog.csdn.net/jeikerxiao/article/details/84105529)以方便之后的学习，如下图所示，此处我创建了一个名为cuc的新用户

  <img src="/img/gateway_ip_add.png" style="zoom:39%;" />
  
  * victim（被攻击者）所需网卡为内部网络，**分为两组(intnet1/2)**，分别在不同的局域网内，即上图后两个IP地址172.16.111.1/24 & 172.16.222.1/24
  
  <img src="/img/victim_natwork.png" style="zoom:39%;" />

  * 攻击用Kali配置NAT网卡，IP地址10.0.2.15/24
  
  <img src="/img/attacker_ip_add.png" style="zoom:38%;" />
  
  
  ​	需要特别注意的是，xp因为系统落后时代过多，配置网卡时默认的Intel千兆网卡芯片没有相关驱动，所以将它改成像下图箭头所示网卡驱动，**不论哪一台虚拟机的网卡都请务必勾上接入网线**
  
  <img src="/img/xp_network.png" style="zoom:38%;" />
  
  

#### 测试网络拓扑连通性

​	重新回看一下我们的要求，不要再往回翻页了：

- 完成以下网络连通性测试；
  - 靶机可以直接访问攻击者主机
  - 攻击者主机无法直接访问靶机
  - 网关可以直接访问攻击者主机和靶机
  - 靶机的所有对外上下行流量必须经过网关
  - 所有节点均可以访问互联网

---

* **靶机可以直接访问攻击者主机**
* victim-kali-1

<img src="/img/kali1_ping_ATT.png" style="zoom:37%;" />

* victim-xp-1

<img src="/img/xp1_ping_ATT.png" alt="xp-1pingATT" style="zoom:39%;" />

* victim-debian-2

  <img src="/img/debian2_ping_ATT.png" style="zoom:38%;" />

* victim-xp-2

  <img src="/img/xp2_ping_ATT.png" style="zoom:38%;" />


---

* **攻击者主机无法直接访问靶机**

* 访问intnet1网络

<img src="/img/ATT_ping_intnet1.png" alt="ATT_ping_intnet1" style="zoom:37%;" />

* 访问intnet2网络

<img src="/img/ATT_ping_intnet2.png" alt="ATT_ping_intnet2" style="zoom:37%;" />

> 如下图，攻击机是可以联网的，说明不是网络问题

<img src="/img/ATT_ping_baidu.png" style="zoom:38%;" />

---

* 网关可以直接访问攻击者主机和靶机，**注意需要关掉xp上的防火墙**，不然会失败

<img src="/img/gateway_ping_ATT.png" style="zoom:39%;" />

<img src="/img/gateway_ping_intnet1.png" alt="gateway ping intnet1" style="zoom:38%;" />

<img src="/img/gateway_ping_intnet2.png" style="zoom:38%;" />

---

* **靶机的所有对外上下行流量必须经过网关**

  * 在网关上使用如下命令

```
sudo tcpdump -i enp0s9 -n
```

​	这里以1组的xp为示例，尝试用其ping一下百度，下图是网关上的监视画面

<img src="/img/gateway_monitor_1.png" style="zoom:38%;" />

---

* **所有节点均可以访问互联网**
* ATT

![](/img/ATT_ping_baidu.png)

* 网关

![gateway_baidu](/img/gateway_baidu.png)

* 1组靶机

![kali1_ping_baidu](/img/kali1_ping_baidu.png)

![xp1_ping_baidu](/img/xp1_ping_baidu.png)

* 2组靶机

<img src="/img/debian2_ping_baidu.png" style="zoom:38%;" />

<img src="/img/xp2_ping_baidu.png" style="zoom:38%;" />



#### 参考链接

[2020-ns-public-shiancuc/blob/ns_chap0x01]: https://github.com/CUCCS/2020-ns-public-shiancuc/blob/ns_chap0x01/ns_chap0x01/ns_chap0x01_report.md
[ 为debian系统创建新root权限用户 ]: https://www.jianshu.com/p/a60340df9004?utm_campaign=
[在debian设置ssh]: https://blog.csdn.net/jeikerxiao/article/details/84105529
[ssh: connect to host localhost port 22: Connection refused 问题]: https://blog.csdn.net/jszhangyili/article/details/8881807

