安装Keepalived
^^^^^^^^^^^^^^

我们可以从各种发行版的软件仓库安装Keepalived，或者从编代码编译安装。虽然从软件仓库进行安装通常是在系统上运行Keepalived的最快方法，\
但软件仓库中可用的Keepalived版本通常是最新的可用稳定版本之后的一些版本。

从软件仓库安装
--------------

在Red Hat Enterprise Linux上安装
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

从 ``Red Hat 6.4`` 开始，Red Hat和克隆版本已经将Keepalived包含在基本的软件仓库中。因此，运行以下命令以使用 ``YUM`` 安装Keepalived包和所有必需的依赖项::

    yum install keepalived

在Debian上安装
>>>>>>>>>>>>>>

运行如下命令以使用 ``Debian`` 的 ``APT`` 包处理实用程序安装Keepalived包和所有必需的依赖项::

    apt-get install keepalived


从源代码编译和构建
------------------

要运行最新的稳定版本，请从源代码编译Keepalived。编译Keepalived需要编译器，``OpenSSL`` 和 ``Netlink`` 库。您可以选择安装 ``SNMP`` 支持所需的 ``Net-SNMP``。

在RHEL/Centos上安装先决条件
>>>>>>>>>>>>>>>>>>>>>>>>>>>

在RHEL上，安装如下先决条件::

    yum install curl gcc openssl-devel libnl3-devel net-snmp-devel

在Debian上安装先决条件
>>>>>>>>>>>>>>>>>>>>>>

在Debian上，安装如下先决条件::

    apt-get install curl gcc libssl-dev libnl-3-dev libnl-genl-3-dev libsnmp-dev

构建和安装
>>>>>>>>>>

使用 ``curl`` 或任何其他传输工具（如 ``wget``）下载Keepalived。该软件可以从 Keepalived官网_ 或 Github仓库_ 获得。然后，进行编译::

    curl --progress http://keepalived.org/software/keepalived-1.2.15.tar.gz | tar xz
    cd keepalived-1.2.15
    ./configure
    make
    sudo make install

.. _Keepalived官网: http://www.keepalived.org/download.html
.. _Github仓库: https://github.com/acassen/keepalived

从源代码编译时一般建议指定PREFIX。例如::

    ./configure --prefix=/usr/local/keepalived-1.2.15

这样，只需删除父目录即可轻松卸载Keepalived的编译版本。此外，这种安装方法允许安装多个版本的Keepalived，而不会互相覆盖。使用符号链接指向所需的版本。例如，您的目录结构可能如下所示::

    [root@lvs1 ~]# cd /usr/local
    [root@lvs1 local]# ls -l
    total 12
    lrwxrwxrwx. 1 root root   17 Feb 24 20:23 keepalived -> keepalived-1.2.15
    drwxr-xr-x. 2 root root 4096 Feb 24 20:22 keepalived-1.2.13
    drwxr-xr-x. 2 root root 4096 Feb 24 20:22 keepalived-1.2.14
    drwxr-xr-x. 2 root root 4096 Feb 24 20:22 keepalived-1.2.15

设置Init脚本
>>>>>>>>>>>>

编译之后，创建一个 ``Init`` 脚本以控制Keepalived守护进程。

在RHEL上::

    ln -s /etc/rc.d/init.d/keepalived.init /etc/rc.d/rc3.d/S99keepalived

在Debian上::

    ln -s /etc/init.d/keepalived.init /etc/rc2.d/S99keepalived

**注意**：该链接应该添加到默认的运行级别目录中。
