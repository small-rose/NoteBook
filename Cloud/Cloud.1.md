---
layout: default
title: Cloud
nav_order: 2
has_children: true
permalink: docs/Cloud
---


# Cloud
{: .no_toc }



This is the development related spring cloud.
{: .fs-6 .fw-300 }


## 微服务的主要技术栈：

|微服务条目	|主流技术 |
|-----------|--------|
|服务开发	|Springboot、Spring、SpringMVC|
|服务配置与管理	|Netflix公司的Archaius、阿里的Diamond等|
|服务注册与发现	|Eureka、Consul、Zookeeper、Nacos等|
|服务调用	|Rest、RPC、gRPC|
|服务熔断器	|Hystrix、sentinel、Envoy等|
|负载均衡	|Ribbon、Nginx等|
|服务接口调用(客户端调用服务的简化工具)	|Feign等|
|消息队列	|Kafka、RabbitMQ、ActiveMQ等 |
|服务配置中心管理	|SpringCloudConfig、Nacos、Chef等 |
|服务路由（API网关）	|Zuul、Geteway等 |
|服务监控	|Zabbix、Nagios、Metrics、Spectator等 |
|全链路追踪	|Zipkin，Brave、Dapper等 |
|服务部署	|Docker、OpenStack、Kubernetes、k3s等 |
|数据流操作开发包	|SpringCloud Stream（封装与Redis,Rabbit、Kafka等发送接收消息） |
|事件消息总线	|Spring Cloud Bus、Nacos等 |



## 微服务组件

0、SpringCloud 微服务
    
 - SpringCloud Eureka 服务注册与发现
 - SpringCloud Consul 服务注册与发现
 - SpringCloud Ribbon 客户端负载均衡与服务调用
 - SpringCloud OpenFeign 服务接口调用
 - SpringCloud Hystrix 熔断器
 - SpringCloud Gateway 服务网关
 - SpringCloud Zuul 服务网关
 - Spring Cloud Config 分布式配置中心
 - Spring Cloud Bus 消息总线
 - SpringCloud Stream 消息驱动
 - SpringCloud Sleuth 分布式请求链路追踪
 - SpringCloud Alibaba
    - SpringCloud Alibaba Nacos服务注册和配置中心
    - SpringCloud Alibaba Sentinel 实现熔断与限流
    - SpringCloud Alibaba Seata 分布式事务


1、常见的服务网关

 - 1.1、Nginx/OpenResty
 - 1.2、Zuul
 - 1.3、SpringCloud Gateway
 - 1.4、Kong。 基于NGINX + Lua 和Apache Cassandra或PostgreSQL构建的
 - 1.5、Traefik

2、常见的注册中心

 - 2.1、Eureka
 - 2.2、Zookeeper
 - 2.3、Consul
 - 2.4、Nacos

3、常见的RPC框架

 - 3.1、Dubbo
 - 3.2、Motan
 - 3.3、Tars
 - 3.4、Spring Cloud
 - 3.5、gRPC
 - 3.6、Thrift
 
