+++
date = "2013-11-13T19:17:16+08:00"
menu = "main"
tags = ["cloudfoundry", "golang", "buildpack", "paas", "docker"]
title = "PaaS要解决的问题"

+++

# 写在前面的话

本来要阐述一下CloudFoundry中各个组件的作用的，感觉应该先做个背景铺垫，就先写写paas要解决的问题，一写写多了，独立成文，之后再续

一句话概括paas解决的问题：**搞定你写好程序之后剩下的事情**

# 站长的烦恼，真正的全栈工程师

比如你现在要写一个小站，代码写好之后你要做什么？

- 去买个VPS
- 去买个域名并设置A记录到你的VPS
- SSH到VPS上开始安装基础环境，比如curl/git/vim/ctags/tree
- SSH到VPS上开始安装运行环境，比如php程序依赖的MySQL/Nginx/PHP/Memcache，Java程序依赖的JDK/Tomcat/MySQL，Python程序的话需要安装MySQLdb/Flask等等，过程中可能会因为网络问题，因为各种lib的版本依赖搞到崩溃
- git clone git@github.com:you/project.git，并编译你的程序
- 配置你的程序，比如MySQL连接地址，Redis连接地址，log的打印的位置
- 配置你的web server，比如tomcat，你要通过配置让tomcat把你的程序run起来

OK，看起来第一阶段搞定了，程序可以跑起来了。但是：你的程序存在各种单点问题，网站稳定性难以得到保证，数据安全难以得到保证，随着流量上扬，你可能会做：

- 把MySQL单独拆出作为一个机器或者多个机器，做一下master-slave集群
- 把tomcat（拿Java程序举例）部署到多台机器上，做一下负载均衡，某一台挂掉了要自动感知立马摘除
- 你的程序可能要写本地文件，比如有个上传头像功能，当你只有一个web server的时候没啥问题，现在牛逼了，server多了，就需要处理这种状态信息，其他状态信息比如session也是一样需要想办法搞定
- cache也一样，比如你使用了redis，为了提高可用性，也需要做集群
- 树大招风，项目刚有起色结果又受到了各种安全攻击，什么DDOS，XSS，SQL注入接踵而至
- ……

过程中，你扮演了各种牛逼的角色，负责研发写代码的开发工程师、负责买机器，装机，配置网络的系统工程师、负责安装维护基础环境/生产环境的运维工程师、负责数据库的DBA、负责防攻击/漏洞检测的安全工程师……而令你更为苦恼的是：当你忙于这些杂七杂八的东西的时候，竞争对手的产品正在快速迭代占领市场！


# 理想的PaaS形态

paas应该解决我们上面的各种问题，对用户的接口应该是这个样子的：

- 提供一个git让用户去push代码，帮助用户做版本化
- 提供一个web portal，让用户看到自己当前创建了哪些app，可以对某些app做升级上线
- 用户点击升级按钮，后台应该自动帮用户去做编译（自动搞定依赖的语言环境和lib库），自动把编译产物推送到线上，可以上线多个实例来防止单点故障
- 如果程序中依赖MySQL/Redis之类的服务，用户只需要提供一个配置文件说明即可。编译的时候自动探测用户的连接配置文件，把地址改成我们云端的DB地址
- 对于写文件的需求，最好利用一些云存储，比如七牛、又拍这种模式，对paas提供商来说，尽量不要挂载NFS之类的文件系统，自找麻烦
- 有很靠谱的容错，某个实例宕掉了，应该可以立马侦测到并启动新的实例，同时收集错误log，发报警给用户
- 探测app使用的CPU/MEMORY情况，如果所有实例都特别耗资源，说明需要扩容了，自动创建新的实例来分担压力
- paas提供商应该提供前端接入层，帮用户做负载均衡和防攻击
- 提供一个统一的二级域名，如果用户需要使用自己的域名，可以通过cname支持
- 各个用户的实例需要跑在container中，做资源隔离，相互之间不能受影响
- 提供dashboard让用户了解自己的app的资源使用情况，pv、uv情况，可以方便的在线查看log
- ……