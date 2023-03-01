# *套餐章节（略）

# 阿里云短信服务

短消息服务SMS(Short Message Service)

应用场景：

- 验证码

- 短信通知

- 推广短信（TD退订）

# Linux

**NAT方式是内部地址，桥接则是与主机同一样网段的IP。**

终端ipconfig查询（虚拟机目录下）

## 命令行终端开启服务器操作

虚拟机安装目录：C:\Program Files\Oracle\VirtualBox

[Oracle VM VirtualBox命令行打开虚拟机并且后台运行 | HiSEN](https://hisen.me/20170219-Oracle-VM-VirtualBox%E5%91%BD%E4%BB%A4%E8%A1%8C%E6%89%93%E5%BC%80%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%B9%B6%E4%B8%94%E5%90%8E%E5%8F%B0%E8%BF%90%E8%A1%8C/)

## XShell连接ubuntu

[xshell连接ubuntu - 腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1454030)

## apt-get失败问题

没网络[Linux運行sudo apt-get update報錯 - 台部落](https://www.twblogs.net/a/5f023fe399927402d4fcfb27)

报错Err连接阿里镜像地址失败。

检查网络问题，重启以下ubuntu就好了。

# SSH连接失败

## apt安装软件报错Could not open lock file /var/lib/dpkg/lock

[linux之安装软件出现Could not open lock file /var/lib/dpkg/lock - open (13: Permission denied)解决总结](https://blog.51cto.com/u_12468355/5095408)

Ubuntu IP inet10.0.2.15

[Xshell 连接 Ubuntu 教程（超详细）,并解决二个常见问题（一直连不上、root用户拒绝密码）](https://blog.csdn.net/qq_44222849/article/details/105043739)

## 22端口连接失败问题

关闭防火墙：

[Xshell 连接 Ubuntu 虚拟机失败](https://blog.csdn.net/qq_45642410/article/details/113785272)

## vi无权修改配置文件问题

[E45: ‘readonly‘ option is set (add ! to override)解决办法_林猛男的博客-CSDN博客](https://blog.csdn.net/longgeaisisi/article/details/107055538)

得用root情况下：wq!强制写入。

先去拿奶茶了

[Xshell 连接 Ubuntu 虚拟机失败_青柠小苍兰的博客-CSDN博客](https://blog.csdn.net/qq_45642410/article/details/113785272)

虚拟机👇

![cmd_7Mnd9D1jw2.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/cmd_7Mnd9D1jw2.png)

`inet 10.0.2.15  netmask 255.255.255.0`

## 主机ping虚拟机失败问题

[Xshell 连接 Ubuntu 虚拟机失败_青柠小苍兰的博客-CSDN博客](https://blog.csdn.net/qq_45642410/article/details/113785272)

## 连接成功

[虚拟机与宿主机网络配置——可互通可上网「建议收藏」 - 腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/2120906)

错误在主机IP和访问虚拟机转发的端口应写本地端口127.0.0.1。计网知识忘完了。常识性基本简单低级错误。
