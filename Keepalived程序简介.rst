Keepalived程序简介
^^^^^^^^^^^^^^^^^^

Keepalived包附带2个程序。

keepalived守护进程
------------------

keepalived命令行参数如下：

.. glossary::

    -f, –use-file=FILE 
        使用指定的配置文件，默认配置文件是“/etc/keepalived/keepalived.conf”。

    -P, –vrrp
        只运行VRRP子系统。这对于不使用IPVS负载均衡的配置很有用。

    -C, –check
        只运行健康检查子系统。这对于将IPVS负载均衡与单个导向器一起使用而没有故障切	换的配置很有用。

    -l, –log-console
        将消息记录到本地控制台。默认行为是将消息记录到syslog。

    -D, –log-detail
        详细的日志信息。

    -S, –log-facility=[0-7]
        设置syslog设施为LOG_LOCAL [0-7]。默认的syslog设施是LOG_DAEMON。

    -V, –dont-release-vrrp
        在守护进程停止时不要删除VRRP VIP和VROUTE。默认行为是在keepalived退出时删除	所有VIP和VROUTE.

    -I, –dont-release-ipvs
        在守护进程停止时不要删除IPVS拓扑。默认行为是在keepalived退出时，它将从IPVS	虚拟服务器表中删除所有条目。

    -R, –dont-respawn
        不要重新派生子进程。默认行为是如果任一进程退出，就重新启动VRRP和checker子	进程。

    -n, –dont-fork
        不要派生守护进程。此选项将导致keepalived在前台运行。

    -d, –dump-conf
        转储配置数据。

    -p, –pid=FILE
        将指定的pidfile文件用于父级keepalived进程。Keepalived的默认pid文件是	“/var/run/keepalived.pid”。

    -r, –vrrp_pid=FILE
        将指定的pidfile文件用于VRRP子进程。VRRP子进程的默认pid文件是	“/var/run/keepalived_vrrp.pid”。

    -c, –checkers_pid=FILE
        将指定的pidfile文件用于checker子进程。checker子进程的默认pid文件是	“/var/run/keepalived_checkers.pid”。

    -x, –snmp
        启用SNMP子系统。

    -v, –version
        显示版本并退出。

    -h, –help
        显示此帮助信息并退出。


genhash实用程序
---------------

genhash二进制文件用于生成只要字符串。genhash的命令行参数如下：

.. glossary::

    –use-ssl, -S
        使用SSL连接到服务器。

    –server <host>, -s
        指定要连接的IP地址。

    –port <port>, -p
        指定要连接的端口。

    –url <url>, -u
        指定要生成哈希值的文件路径。

    –use-virtualhost <host>, -V
        指定要与HTTP请求头(host字段)一起发送的虚拟主机

    –hash <alg>, -H
        指定哈希算法以生成目标页面的摘要。请参阅帮助信息，查看带有默认标记的可用列表。

    –verbose, -v
        输出详细信息。

    –help, -h
        显示此帮助信息并退出。

    –release, -r
        显示版本号并退出。


运行keepalived守护进程
----------------------

要运行keepalived，只需键入::

    [root@lvs tmp]# /etc/rc.d/init.d/keepalived.init start
    Starting Keepalived for LVS:                            [ OK ]

所有的守护进程消息都通过Linux ``syslog`` 进行记录。如果使用“转储配置数据”选项启动keepalived，\
您应该在 ``/var/log/messages`` （在Debian上可能是 ``/var/log/daemon.log``，具体取决于您的syslog配置）中看到如下信息::

    Jun 7 18:17:03 lvs1 Keepalived: Starting Keepalived v0.6.1 (06/13, 2002)
    Jun 7 18:17:03 lvs1 Keepalived: Configuration is using : 92013 Bytes
    Jun 7 18:17:03 lvs1 Keepalived: ------< Global definitions >------
    Jun 7 18:17:03 lvs1 Keepalived: LVS ID = LVS_PROD
    Jun 7 18:17:03 lvs1 Keepalived: Smtp server = 192.168.200.1
    Jun 7 18:17:03 lvs1 Keepalived: Smtp server connection timeout = 30
    Jun 7 18:17:03 lvs1 Keepalived: Email notification from = keepalived@domain.com
    Jun 7 18:17:03 lvs1 Keepalived: Email notification = alert@domain.com
    Jun 7 18:17:03 lvs1 Keepalived: Email notification = 0633556699@domain.com
    Jun 7 18:17:03 lvs1 Keepalived: ------< SSL definitions >------
    Jun 7 18:17:03 lvs1 Keepalived: Using autogen SSL context
    Jun 7 18:17:03 lvs1 Keepalived: ------< LVS Topology >------
    Jun 7 18:17:03 lvs1 Keepalived: System is compiled with LVS v0.9.8
    Jun 7 18:17:03 lvs1 Keepalived: VIP = 10.10.10.2, VPORT = 80
    Jun 7 18:17:03 lvs1 Keepalived: VirtualHost = www.domain1.com
    Jun 7 18:17:03 lvs1 Keepalived: delay_loop = 6, lb_algo = rr
    Jun 7 18:17:03 lvs1 Keepalived: persistence timeout = 50
    Jun 7 18:17:04 lvs1 Keepalived: persistence granularity = 255.255.240.0
    Jun 7 18:17:04 lvs1 Keepalived: protocol = TCP
    Jun 7 18:17:04 lvs1 Keepalived: lb_kind = NAT
    Jun 7 18:17:04 lvs1 Keepalived: sorry server = 192.168.200.200:80
    Jun 7 18:17:04 lvs1 Keepalived: RIP = 192.168.200.2, RPORT = 80, WEIGHT = 1
    Jun 7 18:17:04 lvs1 Keepalived: RIP = 192.168.200.3, RPORT = 80, WEIGHT = 2
    Jun 7 18:17:04 lvs1 Keepalived: VIP = 10.10.10.3, VPORT = 443
    Jun 7 18:17:04 lvs1 Keepalived: VirtualHost = www.domain2.com
    Jun 7 18:17:04 lvs1 Keepalived: delay_loop = 3, lb_algo = rr
    Jun 7 18:17:04 lvs1 Keepalived: persistence timeout = 50
    Jun 7 18:17:04 lvs1 Keepalived: protocol = TCP
    Jun 7 18:17:04 lvs1 Keepalived: lb_kind = NAT
    Jun 7 18:17:04 lvs1 Keepalived: RIP = 192.168.200.4, RPORT = 443, WEIGHT = 1
    Jun 7 18:17:04 lvs1 Keepalived: RIP = 192.168.200.5, RPORT = 1358, WEIGHT = 1
    Jun 7 18:17:05 lvs1 Keepalived: ------< Health checkers >------
    Jun 7 18:17:05 lvs1 Keepalived: 192.168.200.2:80
    Jun 7 18:17:05 lvs1 Keepalived: Keepalive method = HTTP_GET
    Jun 7 18:17:05 lvs1 Keepalived: Connection timeout = 3
    Jun 7 18:17:05 lvs1 Keepalived: Nb get retry = 3
    Jun 7 18:17:05 lvs1 Keepalived: Delay before retry = 3
    Jun 7 18:17:05 lvs1 Keepalived: Checked url = /testurl/test.jsp,
    Jun 7 18:17:05 lvs1 Keepalived: digest = 640205b7b0fc66c1ea91c463fac6334d
    Jun 7 18:17:05 lvs1 Keepalived: 192.168.200.3:80
    Jun 7 18:17:05 lvs1 Keepalived: Keepalive method = HTTP_GET
    Jun 7 18:17:05 lvs1 Keepalived: Connection timeout = 3
    Jun 7 18:17:05 lvs1 Keepalived: Nb get retry = 3
    Jun 7 18:17:05 lvs1 Keepalived: Delay before retry = 3
    Jun 7 18:17:05 lvs1 Keepalived: Checked url = /testurl/test.jsp,
    Jun 7 18:17:05 lvs1 Keepalived: digest = 640205b7b0fc66c1ea91c463fac6334c
    Jun 7 18:17:05 lvs1 Keepalived: Checked url = /testurl2/test.jsp,
    Jun 7 18:17:05 lvs1 Keepalived: digest = 640205b7b0fc66c1ea91c463fac6334c
    Jun 7 18:17:06 lvs1 Keepalived: 192.168.200.4:443
    Jun 7 18:17:06 lvs1 Keepalived: Keepalive method = SSL_GET
    Jun 7 18:17:06 lvs1 Keepalived: Connection timeout = 3
    Jun 7 18:17:06 lvs1 Keepalived: Nb get retry = 3
    Jun 7 18:17:06 lvs1 Keepalived: Delay before retry = 3
    Jun 7 18:17:06 lvs1 Keepalived: Checked url = /testurl/test.jsp,
    Jun 7 18:17:05 lvs1 Keepalived: digest = 640205b7b0fc66c1ea91c463fac6334d
    Jun 7 18:17:06 lvs1 Keepalived: Checked url = /testurl2/test.jsp,
    Jun 7 18:17:05 lvs1 Keepalived: digest = 640205b7b0fc66c1ea91c463fac6334d
    Jun 7 18:17:06 lvs1 Keepalived: 192.168.200.5:1358
    Jun 7 18:17:06 lvs1 Keepalived: Keepalive method = TCP_CHECK
    Jun 7 18:17:06 lvs1 Keepalived: Connection timeout = 3
    Jun 7 18:17:06 lvs1 Keepalived: Registering Kernel netlink reflector
