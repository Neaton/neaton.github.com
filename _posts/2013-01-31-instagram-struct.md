---
layout: post
title: "Instagram:2年10亿美元背后的技术架构"
description: "Instagram:2年10亿美元背后的技术架构"
category: 
tags: 
- Instagram
- 架构
#tagline: ""
---
{% include JB/setup %}

[Instagram](http://instagr.am/)是一款免费照片分享移动应用，目前支持IOS和Android。在1年多的时间里，Instagram发展到140万个用户，1.5亿张图片(几个TB)，仅有3个工程师，以10亿美元的天价被Facebook收购。不得不说，Instagram是业界的一个神话。今天我们就来看看到底是什么样的技术架构支撑着这个10亿美元的公司。

Instagram团队之前发表过一篇文章：[What Powers Instagram:Hundreds of Instances,Dozens of Technologies](http://instagram-engineering.tumblr.com/post/13649370142/what-powers-instagram-hundreds-of-instances-dozens-of)。这篇文章中提到的技术架构堪称经典，很适合初创项目的快速启动。

这个小团队使用了很多不同的技术和策略，保证他们能轻松的应付快速增长带来的压力。他们混合使用SQL、NoSQL，一堆开源项目，和云服务；云服务他们选择了Amazon，他们认为Amazon要比他们自己部署IDC更有优势；用异步队列来串联组件；系统架构在众多的对外API和内部Services之上；数据存储在内存和云端；大部分代码是动态语言；等等等等。非常时髦的一个架构，基于这个架构之上他们得以快速前进，并且保持精简。

那篇文章非常值得一读，有兴趣的同学可以直接看原文。这里我列出一些要点：

* 架构原则:1.保持简单;2.不重复造轮子;3.尽量使用成熟稳定的技术
* 3个工程师
* 大量使用Amazon服务，工程师不用耗费时间自己维护服务器
* 100+个EC2实例用于各种用途
* Ubuntu Linux 11.04("Natty Narwhal")，他们认为这个版本更加稳定
* Amazon的ELB(Elastic Load Balancer)路由请求，ELB后面起了3个Nginx实例
* ELB上关闭了SSL，因为它降低了CPU的使用率
* DNS使用Amazon的Route53
* 25+个Django应用服务运行在High-CPU Extra-Large类型的机器上
* CPU使用比内存使用更加容易达到边界值，所以使用High-CPU Extra-Large的实例来平衡内存和CPU
* [Gunicorn](http://gunicorn.org/)是他们的WSGI服务器。Apache更难配置，更耗CPU
* [Fabric](http://fabric.readthedocs.org/en/1.3.3/index.html)被用来在所有机器上并行执行命令。部署花费更少的时间。
* PostgreSQL(存储用户，照片元数据，标签等)运行在12个Quadruple Extra-Large Memory实例上
* 12个PostgreSQL副本运行在不同的节点上
* PostgreSQL的Master-Replica使用[流复制模式](https://github.com/greg2ndQuadrant/repmgr)，利用EBS的快照来频繁备份
* EBS部署在软件RAID上，使用[mdadm](http://en.wikipedia.org/wiki/Mdadm)以获取适当的IO
* 将所有工作中的数据集存储在内存中，因为EBS每秒磁盘寻道次数有限
* [Vmtouch](http://hoytech.com/vmtouch/vmtouch.c)(轻量级的文件系统缓存诊断工具)被用来管理内存数据，尤其是当[故障转移](https://gist.github.com/1424540)时目标机器没有足够空余的内存
* 用XFS做文件系统，保证数据快照时RAID阵列上数据的一致性
* [Pgbouncer](http://pgfoundry.org/projects/pgbouncer/)用作[PostgreSQL的连接池](http://thebuild.com/blog/)
* 有几个TB的照片存在Amazon S3上
* 用Amazon CloudFront做CDN
* Redis用来支撑feed，活动消息，session系统和[其它服务](http://instagram-engineering.tumblr.com/post/12202313862/storing-hundreds-of-millions-of-simple-key-value-pairs)
* Redis运行在几台Quadruple Extra-Large Memory实例上，偶尔也会做下切分
* Redis也是Master-Replica，副本持久化到磁盘上，并由EBS通过快照备份（这么搞是因为他们发现直接在Master上做dump相当吃力）
* [Geo-Search](http://instagram.com/developer/endpoints/media/#get_media_search)使用Solr，Solr提供的JSON接口也很简单易用
* 6个memcached实例做缓存，因为Amazon Elastic Cache服务并不便宜，mmc客户端使用pylibmc和libmemcached
* Gearman用做:向Twitter,Facebook等平台异步分享照片；新照片发布的通知；feed的反送
* 200个Python进程处理Gearman的任务队列
* [Pyapns](https://github.com/samuraisam/pyapns)处理超过10亿条Apple的push通知，异常稳定
* [Munin](http://munin-monitoring.org/)用做监控和系统度量工具，用[Python-Munin](http://samuelks.com/python-munin/)写了很多图表插件，如每分钟注册人数，每秒钟图片发表数等等
* [Pingdom](http://pingdom.com/)做内部服务的监控
* [PagerDuty](http://pagerduty.com/)用来处理通知和事件
* [Sentry](http://pypi.python.org/pypi/django-sentry)用来做Python的错误报告

以上就是Instagram的博文里面提到的技术要点，怎么样，准备好构建下一个10亿美元的应用了么？

### 相关资料：

* [Storing hundreds of millions of simple key-value pairs in Redis](http://instagram-engineering.tumblr.com/post/12651721845/instagram-engineering-challenge-the-unshredder)
* [Simplifying EC2 SSH Connections](http://instagram-engineering.tumblr.com/post/11399488246/simplifying-ec2-ssh-connections)
* [Sharding & IDs at Instagram](http://instagram-engineering.tumblr.com/post/10853187575/sharding-ids-at-instagram)
* [Membase Cluster on EC2 or Amazon ElastiCache?](http://nosql.mypopescu.com/post/13820225002/membase-cluster-on-ec2-or-amazon-elasticache)
