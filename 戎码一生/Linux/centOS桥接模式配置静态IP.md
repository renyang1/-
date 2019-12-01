# centOS桥接模式配置静态IP

参考：

https://www.jianshu.com/p/aa366daf1df9

https://blog.csdn.net/qq_40024178/article/details/93371760

#### 1. 使用```ipconfig /all```命令查看win10上的网关和IP

![1570844484295](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\1570844484295.png)

#### 2. 在VM上设置桥接网络

![1570867718427](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\1570867718427.png)

#### 3. 打开虚拟机，设置桥接静态IP

执行linux命令```vi /etc/sysconfig/network-scripts/ifcfg-ens33	```编辑文件

![1570956666592](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\1570956666592.png)

**重点参数说明**

```
BOOTPROTO=static  #static，静态ip，而不是dhcp，自动获取ip地址。
DEVICE=ens33  #虚拟机网卡名称。
ONBOOT=yes  #开机启用网络配置。
IPADDR=172.16.2.65  #设置我想用的静态ip地址，要和物理主机在同一网段(前3位一样)，但又不能相同。
NETMASK=255.255.252.0  #子网掩码，同物理机的子网掩码
GATEWAY=172.16.0.1  #网关地址，同物理主机的网关地址
```

#### 4. 修改主机名与网卡启动、网关配置（可以忽略）

```
#查询主机名
hostnamectl
#修改主机名
hostnamectl  set-hostname server02

#配置
$ vi /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=server01
GATEWAY=172.16.0.1

#生效
$ /etc/init.d/network restart
```

**注意：**

/etc/sysconfig/network 不是一个存在的文件，直接使用上面的命令vim这个路径，在编辑模式中输入上面的配置内容即可。

#### 5. 重启网卡

执行命令```service network restart```

#### 6. 测试

1. 在linux中执行以下命令测试与外网连接是否正常：

``` $ ping www.baidu.com ``` 

2. 在win上ping linux的ip，看是否连接成功



