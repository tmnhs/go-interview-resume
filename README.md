# go-interview-resume
<div align=center>
<img src="https://img.shields.io/badge/interview-blue"/>
<img src="https://img.shields.io/badge/resume-lightBlue"/>
<img src="https://img.shields.io/badge/mysql-brightgreen"/>
<img src="https://img.shields.io/badge/redis-green"/>
<img src="https://img.shields.io/badge/computer_network-red"/>
<img src="https://img.shields.io/badge/os-blue"/>
</div>


> 这是本人整理的一些与**Golang后端研发**岗位相关的面试笔记,欢迎大家及时补充
>
> 当然并不局限于Golang研发岗位，笔记中还包括**计算机网络、操作系统、MySQL、Redis、系统设计**等八股文，其他语言岗位的也可以阅读

## 简历
>一些建议：
>
>**简介明了**:保持简历简洁明了，使用清晰的布局和格式，使得信息易于阅读。限制简历长度在一页或两页之内，但必须包含基本的个人信息，比如年龄、性别、电话、邮件以及教育经历
>
>**突出重点**:
>- **项目经验**:如果有相关的项目经验，可以列出项目的名称、时间、描述和您在项目中承担的角色和职责，最好别是那些烂大街的项目(点名某外卖、某论坛等)，如果实在没有，可以参考我个人做的分布式定时任务管理平台[Crony](https://github.com/tmnhs/Crony) ，建议看看源码，不是很难，至于怎么在简历中写，可以参考[程序员推荐简历，简介明了](https://github.com/tmnhs/go-interview-resume/blob/main/resume/%E7%A8%8B%E5%BA%8F%E5%91%98%E6%8E%A8%E8%8D%90%E7%AE%80%E5%8E%86%EF%BC%8C%E7%AE%80%E4%BB%8B%E6%98%8E%E4%BA%86.doc) ，还有怎么在面试过程中介绍这个项目可以参考[项目经历介绍.md](https://github.com/tmnhs/go-interview-resume/blob/main/%E9%A1%B9%E7%9B%AE%E7%BB%8F%E5%8E%86%E4%BB%8B%E7%BB%8D.md)
>- **实习经历**:现在应届生如果没有实习经历真不好找工作了，建议大二或大三的时候找一份实习工作
>

- [130套简历](https://github.com/tmnhs/go-interview-resume/tree/main/resume/130%E5%A5%97%E7%AE%80%E5%8E%86)

- [程序员推荐简历，简介明了](https://github.com/tmnhs/go-interview-resume/blob/main/resume/%E7%A8%8B%E5%BA%8F%E5%91%98%E6%8E%A8%E8%8D%90%E7%AE%80%E5%8E%86%EF%BC%8C%E7%AE%80%E4%BB%8B%E6%98%8E%E4%BA%86.doc)

## 面试
> 整理的一些面试八股文，答案不一定准确，如果感觉不准确的可以自行在网上查找验证
>
> 其中❤表示重点
- [Go语言](https://github.com/tmnhs/go-interview-resume/blob/main/interview/go%E8%AF%AD%E8%A8%80.md)

  Golang面试题，包括Go语言的**基础语法**、**垃圾回收**、**内存管理**、**GMP模型**以及**常见数据结构**(channel、map、select...)的底层原理等

  推荐阅读[地鼠文档](https://www.topgoer.cn/), 可以在里面找到许多与go语言相关的文档

  比如[Go专家编程](https://www.topgoer.cn/docs/gozhuanjia/gogfjhk) 、 [Go语言标准库](https://www.topgoer.cn/docs/golangstandard/golangstandard-1cmks9a4kaj3c) 等都值得阅读

- [代码编程](https://github.com/tmnhs/go-interview-resume/blob/main/interview/%E4%BB%A3%E7%A0%81%E7%BC%96%E7%A8%8B(go%E8%AF%AD%E8%A8%80%E5%AE%9E%E7%8E%B0).md)

  面试过程中面试官可能要求实现的一些代码编程

  比如：

  - 两个协程交替打印10个字母和数字

  - 启动 2个groutine 2秒后取消， 第一个协程1秒执行完，第二个协程3秒执行完

    ...


- [常见算法和模板](https://github.com/tmnhs/go-interview-resume/blob/main/interview/%E5%B8%B8%E8%A7%81%E7%AE%97%E6%B3%95%E5%92%8C%E6%A8%A1%E6%9D%BF.md)

  一些常见算法的模板，比如**KMP、LRU算法、二分法、回溯法、分治法、滑动窗口**等

  推荐阅读[algorithm-pattern](https://greyireland.gitbook.io/algorithm-pattern/) ,是基于Go语言的，阅读此文档可以解决面试中绝大部分算法题

- [MySQL](https://github.com/tmnhs/go-interview-resume/blob/main/interview/MySQL.md)

  MySQL的一些面试题，包括:

  - 存储引擎

  - 索引及其优化

  - 事务(MVCC)和锁

  - 分库分表和主从复制

    ...

- [Redis](https://github.com/tmnhs/go-interview-resume/blob/main/interview/redis.md)

  Redis面试题

  包括**基本的数据类型、过期键的处理策略、持久化、集群、主从和哨兵**等

- [计算机网络](https://github.com/tmnhs/go-interview-resume/blob/main/interview/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C.md)

  计算机网络相关面试题

  比如**网络协议、TCP三次握手、四次挥手、http和https**等

- [操作系统](https://github.com/tmnhs/go-interview-resume/blob/main/interview/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F.md)

  操作系统面试题

  比如**线程、进程以及它们之间如何通信的、多路IO复用、内存**等

- [海量数据高频面试题](https://github.com/tmnhs/go-interview-resume/blob/main/interview/%E6%B5%B7%E9%87%8F%E6%95%B0%E6%8D%AE%E9%AB%98%E9%A2%91%E9%9D%A2%E8%AF%95%E9%A2%98.md)

  在海量数据场景下的一些面试题，比如：

  - 寻找热门查询，300万个查询字符串中统计最热门的10个

  - 在2.5亿个整数中找出不重复的整数，内存空间不足以容纳这2.5亿个整数

  - 在5亿个int找它们的中位数

    ...

- [微服务](https://github.com/tmnhs/go-interview-resume/blob/main/interview/%E5%BE%AE%E6%9C%8D%E5%8A%A1.md)

  微服务场景下的面试题，比如服务治理、熔断和降级等

- [系统设计](https://github.com/tmnhs/go-interview-resume/blob/main/interview/%E7%B3%BB%E7%BB%9F%E8%AE%BE%E8%AE%A1%E6%80%9D%E8%B7%AF.md)

  在某些特定场景下设计的面试题，比如：

  - 分布式ID生成器

  - 短网址系统

  - 定时任务调度器

    ...

- [架构设计](https://github.com/tmnhs/go-interview-resume/blob/main/interview/%E6%9E%B6%E6%9E%84%E8%AE%BE%E8%AE%A1.md)

  与架构设计相关的面试题，比如：

  - 为什么要做多级缓存

  - MQ中间件是如何实现消息可靠性投递的

    ...

  还在更新中...



## 说明

面试问题和答案大部分来自于网络，包括：

- [路人张的面试笔记](https://www.mianshi.online/)

- [地鼠文档](https://www.topgoer.cn/)

- [algorithm-pattern](https://greyireland.gitbook.io/algorithm-pattern/)

- 一些知乎上的文章

- [牛客上的面经](https://www.nowcoder.com/)

- [B站](https://www.bilibili.com/index.html)

- 个人整理

答案不一定准确，欢迎大家提issues或者pull requests进行补充
