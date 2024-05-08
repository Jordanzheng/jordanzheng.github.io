# linux可以ping通但是无法ssh登陆的问题

今天在公司遇到一个奇怪的问题，耗费了大半天才解决。特此总结记录一下解决思路与方法，以便后续能为更快定位类似的问题。  



### 问题现象：

无法登陆公司小网中的一台虚拟机，该虚拟机有两张网卡，分别配有两个网段的IP。其中一个网段的IP可以ping通，但是无法ssh登陆，显示**ssh: connect to host 192.171.25.101 port 22: Connection refused**，另外一个网段的IP可以ssh登陆。  


<!-- more -->

### 问题定位过程：

刚开始定位思路是检查ssh服务是否ok？，检查是否打开防火墙？后来转念一想，这台虚拟机都可以通过另外一个网卡IP登陆，说明ssh服务是正常的。后来上网google一下遇到的现象，有篇[英文文章](https://askubuntu.com/questions/30080/how-to-solve-connection-refused-errors-in-ssh-connection)提到可能是IP冲突导致ssh登陆不了。按照提示用**arping**命令检查一下有问题的IP是否冲突，果不其然，真的是冲突了。至此问题原因清楚了。IP冲突导致可以ping通该IP但是无法通过该IP ssh登陆虚拟机。     



### 问题解决办法：

#### 一、检测linux下IP冲突的命令  

**arping**：在IP冲突的同网段其他linux上执行**arping**命令检测是否有IP冲突。

> [root@dev ~]# arping  -I eth0 192.168.9.120
>
> ARPING 192.168.1.120 from 192.168.9.200 eth0
>
> Unicast reply from 192.168.9.120 [40:F4:EC:76:79:C2] 3.084ms
>
> Unicast reply from 192.168.9.120 [50:7B:9D:25:29:59] 0.817ms
>
> Unicast reply from 192.168.9.120 [50:7B:9D:25:29:59] 0.810ms

如果只检查出一个MAC地址，则表示网内A机器的的IP：192.168.9.120是唯一的

如果有以上信息即查出两个MAC地址，则表示网内有一台MAC地址为40:F4:EC:76:79:C2的主机IP地址与A机器相同。

这时可以通过ifconfig命令验证A机器，如下发现：A机器的MAC地址是50:7B:9D:25:29:59 。
我们可以用局域网扫描软件找到MAC地址为40:F4:EC:76:79:C2的主机，并将其隔离或更换IP地址。

**检验原理**：

arping命令是以广播地址发送arp packets，以太网内所有的主机都会收到这个arp packets，但是本机收到之后不会Reply任何信息。

当我们在linux主机端上执行下面的命令时：
arping 192.168.9.120　　
会默认使用eth0，向局域网内所有的主机发送一个：
who has 192.168.9.120的arp request，tell 192.168.9.120 your mac address，

当这台windows主机端收到这个arp packets后，则会应答："I am 192.168.9.120 , mac是00:25:e4:6a:4b:f4"，这样我们会收到mac地址为00:25:e4:6a:4b:f4的windows主机的Reply信息。  

#### 二、修改linux主机IP

修改冲突的IP地址，冲突的网卡是eth0，

1. 修改/etc/sysconfig/network/ifcfg-eth0 的IP
2. 重启网络  service sshd restart

### 问题引申：

IP冲突除了出现上述能ping通但是ssh登陆不了的现象之外，还有可能出现一会儿能登陆、一会儿登陆不了，或者登陆的主机名偶尔会变。其原因可能是 **IP地址冲突后，ssh 登录的设备并不是同一个设备，或者意外的不是你想要登录的设备，因此ssh登录时提示用户验证，但提示信息可能不一样（因为设备不一样了）。**


