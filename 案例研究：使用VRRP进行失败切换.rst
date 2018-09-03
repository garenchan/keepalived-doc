案例研究：使用VRRP进行失败切换
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

作为示例，我们可以引入如下LVS拓扑：

架构描述
--------

要使用VRRPv2协议创建虚拟LVS转发器，我们定义如下架构：

- 2个处于主动-主动配置的LVS转发器。

- 每个LVS转发器4个VRRP实例：2个处于MASTER状态的VRRP实例和2个处于BACKUP状态的VRRP实例。我们在每个LVS转发器上使用对称状态。

- 2个处于相同状态的VRRP实例将进行同步，以定义持久化虚拟路由路径。

- 强认证：IPSEC-AH用于保护我们的VRRP通告免受欺骗和回复攻击。

VRRP实例与以下IP地址混合：

- VRRP实例VI_1：拥有VRRP VIPs VIP1和VIP2。此实例在LVS转发器1上默认为MASTER状态。它与VI_2保持同步。

- VRRP实例VI_2：拥有DIP1。此实例在LVS转发器1上默认为MASTER状态。它与VI_1保持同步。

- VRRP实例VI_3：拥有VRRP VIPs VIP3和VIP4。此实例在LVS转发器2上默认为MASTER状态。它与VI_4保持同步。

- VRRP实例VI_4：拥有DIP2。此实例在LVS转发器2上默认为MASTER状态。它与VI_3保持同步。


Keepalived配置
--------------

整个配置在 ``/etc/keepalived/keepalived.conf`` 文件中完成。在我们的案例研究中，LVS转发器1上的该文件如下所示::

    vrrp_sync_group VG1 {
        group {
            VI_1
            VI_2
        }
    }
    vrrp_sync_group VG2 {
        group {
            VI_3
            VI_4
        }
    }
    vrrp_instance VI_1 {
        state MASTER
        interface eth0
        virtual_router_id 51
        priority 150
        advert_int 1
        authentication {
            auth_type AH
            auth_pass k@l!ve1
        }
        virtual_ipaddress {
            192.168.200.10
            192.168.200.11
        }
    }
    vrrp_instance VI_2 {
        state MASTER
        interface eth1
        virtual_router_id 52
        priority 150
        advert_int 1
        authentication {
            auth_type AH
            auth_pass k@l!ve2
        }
        virtual_ipaddress {
            192.168.100.10
        }
    }

::

    vrrp_instance VI_3 {
        state BACKUP
        interface eth0
        virtual_router_id 53
        priority 100
        advert_int 1
        authentication {
            auth_type AH
            auth_pass k@l!ve3
        }
        virtual_ipaddress {
            192.168.200.12
            192.168.200.13
        }
    }
    vrrp_instance VI_4 {
        state BACKUP
        interface eth1
        virtual_router_id 54
        priority 100
        advert_int 1
        authentication {
            auth_type AH
            auth_pass k@l!ve4
        }
        virtual_ipaddress {
            192.168.100.11
        }
    }

然后我们在LVS转发器2上定义对称配置文件。这意味着LVS转发器2上的VI_3和VI_4处于MASTER状态，具有更高的优先级150且从稳定状态开始。\
LVS转发器2上的VI_1和VI_2处于默认的BACKUP状态，优先级较低为100。此配置文件为每个物理网卡指定2个VRRP实例。当您在LVS转发器1上运行Keepalived\
而不在LVS转发器2上运行它时，LVS转发器1将拥有所有VRRP VIP。因此，如果您使用ip使用程序（在Debian上，ip实用程序是iprouter的一部分），您可能会看到类似的东西::

    [root@lvs1 tmp]# ip address list
    1: lo: <LOOPBACK,UP> mtu 3924 qdisc noqueue
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 brd 127.255.255.255 scope host lo
    2: eth0: <BROADCAST,MULTICAST,UP> mtu 1500 qdisc pfifo_fast qlen 100
        link/ether 00:00:5e:00:01:10 brd ff:ff:ff:ff:ff:ff
        inet 192.168.200.5/24 brd 192.168.200.255 scope global eth0
        inet 192.168.200.10/32 scope global eth0
        inet 192.168.200.11/32 scope global eth0
        inet 192.168.200.12/32 scope global eth0
        inet 192.168.200.13/32 scope global eth0
    3: eth1: <BROADCAST,MULTICAST,UP> mtu 1500 qdisc pfifo_fast qlen 100
        link/ether 00:00:5e:00:01:32 brd ff:ff:ff:ff:ff:ff
        inet 192.168.100.5/24 brd 192.168.201.255 scope global eth1
        inet 192.168.100.10/32 scope global eth1
        inet 192.168.100.11/32 scope global eth1

然后只需在LVS转发器2上启动Keepalived，您将看到::

    [root@lvs1 tmp]# ip address list
    1: lo: <LOOPBACK,UP> mtu 3924 qdisc noqueue
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 brd 127.255.255.255 scope host lo
    2: eth0: <BROADCAST,MULTICAST,UP> mtu 1500 qdisc pfifo_fast qlen 100
        link/ether 00:00:5e:00:01:10 brd ff:ff:ff:ff:ff:ff
        inet 192.168.200.5/24 brd 192.168.200.255 scope global eth0
        inet 192.168.200.10/32 scope global eth0
        inet 192.168.200.11/32 scope global eth0
    3: eth1: <BROADCAST,MULTICAST,UP> mtu 1500 qdisc pfifo_fast qlen 100
        link/ether 00:00:5e:00:01:32 brd ff:ff:ff:ff:ff:ff
        inet 192.168.100.5/24 brd 192.168.201.255 scope global eth1
        inet 192.168.100.10/32 scope global eth1

在LVS转发器2中你会看到对称的::

    [root@lvs2 tmp]# ip address list
    1: lo: <LOOPBACK,UP> mtu 3924 qdisc noqueue
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 brd 127.255.255.255 scope host lo
    2: eth0: <BROADCAST,MULTICAST,UP> mtu 1500 qdisc pfifo_fast qlen 100
        link/ether 00:00:5e:00:01:10 brd ff:ff:ff:ff:ff:ff
        inet 192.168.200.5/24 brd 192.168.200.255 scope global eth0
        inet 192.168.200.12/32 scope global eth0
        inet 192.168.200.13/32 scope global eth0
    3: eth1: <BROADCAST,MULTICAST,UP> mtu 1500 qdisc pfifo_fast qlen 100
        link/ether 00:00:5e:00:01:32 brd ff:ff:ff:ff:ff:ff
        inet 192.168.100.5/24 brd 192.168.201.255 scope global eth1
        inet 192.168.100.11/32 scope global eth1

所用到的VRRP VIP如下：

- VIP1 = 192.168.200.10
- VIP2 = 192.168.200.11
- VIP3 = 192.168.200.12
- VIP4 = 192.168.200.13
- DIP1 = 192.168.100.10
- DIP2 = 192.168.100.11

使用VRRP关键字 ``sync_instance`` 意味着我们已经为每个LVS转发器（VI_1,VI_2）和（VI_3，VI_4）定义了一对MASTER VRRP实例。\
这意味着如果LVS转发器1上的eth0失败，那么LVS转发器2上的VI_1进入MASTER状态，因此两个转发器上的MASTER实例分布将是：\
转发器1的（VI_2）和转发器2上的（VI_1,VI_3和VI_4）。使用 ``sync_instance``，因此LVS转发器1上的VI_2将被强制进入BACKUP状态。\
最终的VRRP MASTER实例分布将是：LVS转发器1的（无）和LVS转发器2的（VI_1,VI_2,,VI_3和VI_4）。如果LVS转发器1上的eth0成为可用时，分布将转换回初始状态。

有关此状态转换的更多信息，请参阅 ``使用VRRPv2的Linux虚拟服务器的高可用性`` （可从http://www.linux-vs.org/~acassen/获得）一文，该文章解释了此功能的实现。

使用此配置，两个LVS转发器都处于主动状态，因此为全局转发器共享LVS转发器。这样我们就引入了虚拟LVS转发器。

.. Note:: 此VRRP配置示例演示了高可用性路由器（不是特定于LVS的路由器）。它可以用于许多更常见/简单的需求。
