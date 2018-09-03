配置SNMP支持
^^^^^^^^^^^^

Keepalived提供了一个 ``SNMP`` 子系统，可以收集有关 ``VRRP`` 堆栈和健康检查系统的各种指标。Keepalived ``MIB`` 位于项目的doc目录中。\
MIB的基本SNMP ``OID`` 是 ``.1.3.6.1.4.1.9586.100.5``，它由 ``IANA`` 分配的 `Debian OID空间`_ 托管。

.. _Debian OID空间: https://dsa.debian.org/iana/

先决条件
--------

在系统上安装SNMP协议工具和库。这需要安装几个包::

    yum install net-snmp net-snmp-utils net-snmp-libs

在您的系统上安装了SNMP后，请在配置keepalived时开启SNMP支持。编译keepalived时，添加--enable-snmp配置选项。例如::

    ./configure --enable-snmp

在编译过程的配置步骤中，您将在使用make构建之前获得配置摘要。例如，您可能会在Centos 6机器上看到类似的输出::

    ./configure --prefix=/usr/local/keepalived-1.2.15 --enable-snmp
    Keepalived configuration
    ------------------------
    Keepalived version       : 1.2.15
    Compiler                 : gcc
    Compiler flags           : -g -O2 -I/usr/include/libnl3
    Extra Lib                : -Wl,-z,relro -Wl,-z,now -L/usr/lib64
    -lnetsnmpagent -lnetsnmphelpers -lnetsnmpmibs -lnetsnmp -Wl,-E
    -Wl,-rpath,/usr/lib64/perl5/CORE -lssl -lcrypto -lcrypt  -lnl-genl-3 -lnl-3
    Use IPVS Framework       : Yes
    IPVS sync daemon support : Yes
    IPVS use libnl           : Yes
    fwmark socket support    : Yes
    Use VRRP Framework       : Yes
    Use VRRP VMAC            : Yes
    SNMP support             : Yes
    SHA1 support             : No
    Use Debug flags          : No

请注意配置摘要的 ``Extra Lib`` 部分。它列出了gcc将用于构建keepalived的各种库标志，其中一些与SNMP有关。


配置支持
--------

通过在SNMP守护进程配置文件中包含以下行来启用 ``SNMP AgentX`` 支持，如果您在Centos 6机器上通过RPM进行安装，那么通常配置文件为 ``/etc/snmp/snmpd.conf``::

    master agentx

.. Note:: 请务必reload或重启SNMP服务，以确保配置更改生效。


添加MIB
-------

您可以使用OID查询keepalived SNMP托管对象。例如::

    snmpwalk -v2c -c public localhost .1.3.6.1.4.1.9586.100.5.1.1.0
    SNMPv2-SMI::enterprises.9586.100.5.1.1.0 = STRING: "Keepalived v1.2.15 (01/10,2015)"

或者，使用keepalived MIB，您可以使用项目中提供的MIB进行查询。首先，将MIB复制到系统的全局MIB目录或用户的本地MIB目录::

    cp /usr/local/src/keepalived-1.2.15/doc/KEEPALIVED-MIB /usr/share/snmp/mibs
    或者
    cp /usr/local/src/keepalived-1.2.15/doc/KEEPALIVED-MIB ~/.snmp/mibs

SNMP守护进程将检查两个目录中是否存在MIB。一旦MIB到位，SNMP查询可以如下所示::

    snmpwalk -v2c -c public localhost KEEPALIVED-MIB::version
    KEEPALIVED-MIB::version.0 = STRING: Keepalived v1.2.15 (01/10,2015)


MIB概述
-------

Keepalived MIB有四个主要部分：

.. hlist::
    :columns: 1
    
    - global
    - vrrp
    - check
    - conformance

Global
>>>>>>

Global部分包含有关keepalived实例信息的对象，例如版本，路由器ID和管理电子邮箱。

VRRP
>>>>

VRRP部分包含每个已配置的VRRP实例信息的对象。在每个实例中，都有包含实例名称，当前状态和虚拟IP地址的对象。

Check
>>>>>

Check部分包含有关每个已配置虚拟服务器信息的对象。它包括用于虚拟和真实服务器的服务器表，还包括配置的负载均衡算法，\
负载均衡方法，协议，状态，真实和虚拟服务器网络连接统计信息。

Comformance
>>>>>>>>>>>

.. Todo:: 完善comformance

.. Note:: 使用MIB浏览器（如mbrowse）查看可用于查询的托管对象，以监控LVS服务器的运行状况。
