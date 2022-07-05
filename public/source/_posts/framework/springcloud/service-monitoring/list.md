---
title: springcloud 服务监控系统盘点
categories: springcloud
tags:
  - springcloud
  - APM
abbrlink: 79a1e372
date: 2022-02-02 12:00:00
---
服务监控相关系统

APM 系统（Application Performance Management，即应用性能管理）是对企业的应用系统进行实时监控，实现对应用性能管理和故障定位的系统化解决方案，在运维中常用。

- SpringBoot Admin

  SpringCloud配置软件，能够将 Actuator 中的信息进行界面化的展示，也可以监控所有 Spring Boot 应用的健康状况，提供实时警报功能。

- CAT（开源）：https://github.com/dianping/cat

  由国内美团点评开源的，基于 Java 语言开发，目前提供 Java、C/C++、Node.js、Python、Go 等语言的客户端，监控数据会全量统计。国内很多公司在用，例如美团点评、携程、拼多多等。CAT 需要开发人员手动在应用程序中埋点，对代码侵入性比较强。

- Zipkin（开源）：https://github.com/openzipkin/zipkin

  由 Twitter 公司开发并开源，Java 语言实现。侵入性相对于 CAT 要低一点，需要对web.xml 等相关配置文件进行修改，但依然对系统有一定的侵入性。Zipkin 可以轻松与 Spring Cloud 进行集成，也是 Spring Cloud 推荐的 APM 系统。比如（[Sleuth+Zipkin 链路跟踪](framework/spring-cloud/service-link-tracking/sleuth)）。

- Pinpoint（开源） ：https://github.com/pinpoint-apm/pinpoint

  韩国团队开源的 APM 产品，运用了字节码增强技术，只需要在启动时添加启动参数即可实现 APM 功能，对代码无侵入。目前支持 Java 和 PHP 语言，底层采用 HBase 来存储数据，探针收集的数据粒度非常细，但性能损耗较大，因其出现的时间较长，完成度也很高，文档也较为丰富，应用的公司较多。

- SkyWalking（开源）：https://github.com/apache/skywalking

  国人开源的产品，2019 年 4 月 17 日 SkyWalking 从 Apache 基金会的孵化器毕业成为顶级项目。目前 SkyWalking 支持 Java、.Net、Node.js 等探针，数据存储支持MySQL、ElasticSearch等。

- Elastic APM：

  基于Elastic Stack构建的应用性能监控（APM）系统。



---



**什么是APM系统?**

　　APM系统可以帮助理解系统行为、用于分析性能问题的工具，以便发生故障的时候，能够快速定位和解决问题，这就是APM系统，全称是(Application Performance Monitor)。

　　谷歌公开的论文提到的 Google Dapper 可以说是最早的 APM 系统了，给 google 的开发者和运维团队帮了大忙，所以谷歌公开论文分享了 Dapper。

　　而后，很多的技术公司基于这篇论文的原理，设计开发了很多出色的 APM 框架，例如 Pinpoint、SkyWalking 等。

　　而 SpringCloud 官方集成了一套这样的系统：Spring Cloud Sleuth + Zipkin。



[「dapper 分布式跟踪系统论文原文.pdf」](https://www.aliyundrive.com/s/2UWa2k6t2uC )

[「翻译的版本.pdf」](https://bigbully.github.io/Dapper-translation/)

