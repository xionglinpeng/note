# 微服务结构~携程Apollo配置中心结构剖析【跳转[原文](<https://mp.weixin.qq.com/s/-hUaQPzfsl9Lm3IqQW3VDQ>)】

一、介绍

Apollo（阿波罗）是携程框架部研发并开源的一款生产级的配置中心产品，它能够集中管理应用在不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限，流程治理等特性，适用于微服务配置管理场景。

Apollo目前在国内开发者社区比较热，在Github上有超过5k颗星，在国内众多互联网公司有落地案例，可以说Apollo是目前配置中心产品领域Number 1的产品，其成熟度和企业级特性要远远强于Spring Cloud体系中的Spring Cloud Config产品。

Apollo采用分布式微服务架构，它的架构有一点复杂，Apollo的作者宋顺虽然给出了一个机构图。但是如果没有一定的分布式微服务架构基础的话，则普通的开发人员甚至是架构师也很难一下子理解。只有完全理解了Apollo的架构，大家才能在生产实践中更好的部署和使用Apollo。另外，通过学习Apollo的架构，大家可以深入理解微服务架构的一些基本原理。

二、架构和模块

下面Apollo官方架构图

![](https://mmbiz.qpic.cn/mmbiz_png/ELH62gpbFmGdnIjxDT7AOQyZgl2KQnz6LCwSGeZjrh5DlMd0MMxVIepCFQKdE6vfJWbZOKiaHqEcmia1nJia2o7Vg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如果没有足够的分布式微服务架构基础的话，对携程的一些框架产品（比如Software Load Balancer（SLB））

