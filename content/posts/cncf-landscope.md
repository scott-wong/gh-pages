---
title: "企业架构系列：从CNCF云原生全景图里做技术选型"
date: 2020-04-14T17:54:08+08:00
draft: false
categories: ["架构", "技术架构"]
tags: ["云原生", "CNCF"]
---

# 概述
从全局视角了解云原生生态可以直接看[CNCF全景图](https://landscape.cncf.io/)，截止2020年4月14日已有1,373个图标。包含应用定义与开发层（App Definition and Development）、编排与治理层（Orchestration Management）、运行时（Runtime）、供应保障层（Provisioning）、平台（Platform）、观察与分析（Observability and Analysis）、无服务（Serverless）、云服务商（Special）和CNCF成员（Members）八个层次。  
![CNCF全景图](/images/cncf-landscope.png)  
从CNCF全景图里做技术选型是个比较靠谱的方法，具体还是根据客户需要或行业特点来选。
本文重点介绍图中涉及的开源项目。

# 目录
[1.应用定义与开发层](#1应用定义与开发层)  
[2.编排与治理层](#2编排与治理层)  
[3.运行时](#3运行时)  
[4.供应保障层](#4供应保障层)  
[5.平台](#5平台)  
[6.观察与分析](#6观察与分析)  
[7.Serverless](#7serverless)  
[8.云服务商（Special）](#8云服务商special)  
[9.CNCF成员](#9cncf成员) 

---

# 1.应用定义与开发层
本层包括数据库、流与消息、应用定义和镜像构建、持续集成/部署四部分。
## 1.1 数据库
CNCF包含了业界主流的所有数据库，比如关系型Oracle、MySQL、PostgreSQL、MS SQL Server，支持MySQL分片的中间件Vitess、国产HTAP的TiDB，云原生CockroachDB，分析型Hadoop，文档MongoDB，内存数据库Redis、Ignite(国内用的不多，有坑)，图Neo4j等47个。  
![CNCF全景图-数据库](/images/cncf-landscope-database.png)  

选型建议：
- 客户接受公有云的直接用云厂商的数据库产品，如阿里的NewSQL PolarDB(支持最大容量100TB，最多可横向扩展16个节点，每个节点最高88 vCPU)，OceanBase(单集群最大数据量超过 3PB，最大单表行数达万亿级)；
- 很多行业需要独立部署的，商业数据库Oracle等仍然占有一席之地，而且Oracle也进化成自治数据库，仍然可圈可点；
- 客户要独立部署但接受新事物的，大胆使用支持行存+列存的HTAP混合的TiDB，高度兼容MySQL，3.0集群支持 150+ 存储节点，300+TB存储容量；
- 内存数据库Redis足够稳定，官方集群部署方案能适应大多场景；
- 小型应用场景可以直接Helm安装MySQL用。

## 1.2 流与消息
![CNCF全景图-流与消息](/images/cncf-landscope-message.png)  
流计算引擎目前基本是Spark和Flink二分天下，批处理Spark更强大些，流处理Flink前期优势明显，目前Spark也在紧跟，双方也是互相借鉴。Spark生态支持更好些，按需二选一就可以了。

消息中间件目前国内Kafka、RocketMQ、RabbitMQ用的比较多，各有适用场景。
Kafka在大数据流处理、日志等场景用的多，RocketMQ在国内通用消息队列场景有很多落地，RabbitMQ在企业级市场也广泛采用。

分布式消息标准目前有国内牵头的OpenMessaging在推进。

CNCF有两个孵化项目CloudEvents和NATS。
- CloudEvents是描述事件数据的开源规范，旨在简化事件声明以及跨服务、平台等的消息投递。
- NATS是高性能消息中间件，官网性能对比图是Kafka两倍，目前用的不多，可持续关注。

这里还包括了几个数据集成/ETL工具开源的NiFi和商业的Talend等，我们用的是图里面没有的Pentaho Data Integration (Kettle)。

最后提一下MQTT的消息中间件EMQ X，国内公司开源的，在物联网有广泛应用。

## 1.3 应用定义和镜像构建  
![CNCF全景图-应用定义和镜像构建](/images/cncf-landscope-app-definition.png)  

CNCF孵化中的Helm几乎成为云原生应用定义的事实标准，推荐企业在内部维护一套基于Helm的应用商店，同时国内可以用阿里云apphub应用仓库。期待阿里主导的OAM应用定义规范的后续发展。

OpenAPI规范在这层，是API的统一标准，目前版本3.0，建议采用。

Podman，新的容器管理工具，可替代Docker，目前国内用的不多。

镜像构建：
- kaniko，支持在Kubernetes中构建容器镜像。
- Packer，用于从单一配置来源为多平台创建相同的机器映像。
建议可以用Kubesphere集成的Jenkins构建镜像。

Telepresence是比较有用的工具，用于本地开发连接k8s集群。

比较奇怪的是华为开源的ServiceComb微服务框架放在了这层。

## 1.4 持续集成/部署
![CNCF全景图-CI/CD](/images/cncf-landscope-cicd.png)  

CI/CD工具图里面常用的Jenkins + Gitlab，开源项目用Travis CI。  
建议同1.3.

# 2.编排与治理层
编排与治理层（Orchestration Management）由计划与编排(Scheduling & Orchestration)、协调与服务发现(Coordination & Service Discovery)、远程过程调用(Remote Procedure Call)、服务代理(Service Proxy)、API网关、服务网格(Service Mesh)等六部分组成。

## 2.1 计划与编排
作为CNCF毕业项目的Kubernetes已经成为容器编排事实标准，Amazon ECS仍然有一定市场，其他Mesos、Docker Swarm等则正渐行渐远。

## 2.2 协调与服务发现
毕业项目CoreDNS是k8s默认DNS服务器，支持插件扩展。

全景图里服务发现有etcd、Zookeeper、Nacos、Netflix Eureka（已停止开源版本开发）。
- etcd在k8s中用于保存集群所有的网络配置和对象的状态信息，作为KV数据库也应用于众多产品，如网关Apisix。
- Zookeeper，早期dubbo+zk是经典的服务架构，目前国内应用仍然广泛，同时作为Kafka的依赖组件。
- Nacos，阿里开源的服务发现、配置管理和服务管理平台，建议广泛采用。

建议小型微服务项目直接用k8s的Service实现服务发现，中型则可尝试Nacos，或者直接用服务网格。

## 2.3 RPC
RPC框架目前有孵化的gRPC、Thrift、Dubbo、SOFARPC、Tars、Avro，一半国人推出的。国内用的最多的是Dubbo，中小型项目直接RESTful API调用，也是一种RPC方式。

## 2.4 服务代理
毕业项目Envoy作为数据平面与Istio作为服务网格黄金组合正朝着网格标准走去，蚂蚁金服造了个基于Go的轮子MOSN。

其他主要是负载均衡工具，包括F5、HAProxy、Nginx及其衍生产品（Openresty、Tengine）等。

Traefik是云原生边缘路由，未深入研究不做评价。

负载均衡选型建议：  
多数场景Nginx能够胜任，支持四层和七层代理，与Keepalived组成多主模式。如果想实现动态配置，可以直接用基于Openresty的API网关。

## 2.5 API网关
API网关开源里面主要有Openresty和Go两大阵营。
- Kong，基于Openresty，发展有几年了，生态丰富，持久化支持C。
- Apisix，基于Openresty，国人主导的，对标Kong，还在Apache孵化中，持久化用的是etcd。
- Ambassador, 基于Envoy proxy构建的，k8s原生的开源微服务网关。
- Tyk，基于Go，了解的不多，国内有基于Go的GOKU，虽然开源，单多数插件收费。
- Mule Community Edition，国内很多ESB平台是基于MuleESB建设的。
- WSO2 API Microgateway，了解的不多。

选型建议：  
Apisix目前在高速迭代，虽然还有些问题，但已经生产可用了。
另外需要独立提到的流控防护组件Sentinel，产品成熟度高，建议与API网关二选一。

![API网关选型](/images/api-gateway.jpg)
图片来源：Apisix作者之一，图中Tyk拼错了  

## 2.6 服务网格
这里主要是控制平面的产品，虽然Linkerd是CNCF孵化的项目，不过目前还是Istio更具优势，尤其1.5版本发布后，很大程度解决了性能问题。

这里的Netflix Zuul感觉应该移到API网关层，Consul更适合放在协调与服务发现层与Nacos并列。

# 3.运行时
运行时（Runtime）包括、存储、容器运行时和网络三部分。  

## 3.1 云原生存储
![云原生存储](/images/cncf-landscope-storage.png)  

上图可看到众多的存储方案/产品，常见的Ceph、GlusterFS、MinIO。
- Container Storage Interface (CSI)，旨在为容器编排引擎和存储系统间建立一套标准的存储调用接口，通过该接口为容器编排引擎提供存储服务。各个云厂商都有提供CSI接口。
- Ceph，兼有块存储、文件存储功能的高扩展对象存储，支持作为k8s持久化存储；
- GlusterFS，兼有对象存储功能的高扩展文件存储，支持作为k8s持久化存储；
- Rook,一个自管理的分布式存储编排系统，可以为k8s提供便利的存储解决方案。Rook本身并不提供存储，而是在kubernetes和存储系统之间提供适配层，简化存储系统的部署与维护工作，目前支持的存储中Stable状态有Ceph、EdgeFS。
- MinIO，支持对象存储，兼容S3 API，可以用于分布式图片存储。
- Alluxio，以内存为中心的虚拟的分布式存储系统，作为数据访问层成为持久存储层（如Amazon S3，Microsoft Azure Object Store，Apache HDFS或OpenStack Swift）和计算框架层（如Apache Spark，Presto或Hadoop MapReduce）之间的桥梁。


选型建议：
开发测试环境可以用传统的NFS，生产环境建议搭建高可用分布式存储集群，具体根据业务特点和人员专长选择。

## 3.2 容器运行时

容器运行时除了Docker默认的containerd，还有Kata、gVisor(Google)、Firecracker(AWS)、Nabla、Pouch(Ali)等。
- 容器运行时接口(Container Runtime Interface, CRI)，定义容器和镜像的服务的接口。
- containerd，CNCF毕业项目，Docker公司捐赠的。
- CRI-O，能够让k8s使用任意兼容OCI的运行时作为运行pod的容器运行时，默认用runC，还支持其他OCI兼容，例如Kata容器。
- 安全/沙箱容器  
基于MicroVM：Kata、Firecracker  
基于进程虚拟化：gVisor  

各容器引擎各有优势，云原生落地初期建议直接用containerd。

## 3.3 云原生网络
![云原生网络](/images/cncf-landscope-network.png)  
常见网络方案Flannel、Calico、Weave和Canal（技术上是多个插件的组合）。  
- Container Network Interface (CNI)，用于连接容器管理系统和网络插件。

选哪个CNI插件？  
![选哪个CNI插件？](/images/which-cni.webp)  
- Overlay插件，常见的有 Flannel-vxlan, Calico-ipip, Weave；
- Underlay或者路由插件，clico-bgp, flannel-hostgw, sriov；
- 在公有云选择厂商提供的插件。

# 4.供应保障层
供应保障层（Provisioning）包括自动化与配置、容器仓库、安全与合规、密钥管理四部分。

## 4.1 自动化与配置
![自动化与配置](/images/cncf-landscope-automation.png)  
自动化：
- Ansible，自动化运维工具，使用广泛。
- Chef，一款自动化服务器配置管理工具。
- Terraform，"Write, Plan, and create Infrastructure as Code".
- Pulumi，架构即是代码，使用代码自动配置和管理您的AWS，Azure，Google Cloud Platform和/或Kubernetes资源。

配置：
- Apollo，携程开源的强大的配置平台。

KubeEdge，k8s原生的边缘计算框架。

本层研究较少，不多做介绍。

## 4.2 容器仓库
容器镜像仓库国内可以用阿里容器镜像服务或者用CNCF孵化的Harbor自建。  
Dragonfly是阿里开源的P2P镜像分发系统，国内用户较多，类似Uber开源的Kraken。

## 4.3 安全与合规
![安全与合规](/images/cncf-landscope-security.png) 
镜像扫描：
- Anchore，利用 CVE 数据库来对已知威胁进行扫描，还提供很多附加标准可以进行配置，来作为扫描策略的一部分：Dockerfile 检查、凭据泄露、语言相关内容（npm、maven 等）、软件许可等。
- Clair，专注于威胁检测和 CVE 匹配的功能，用户量大，Harbor已集成。
- Dagda，图中没有，集成了 ClamAV，不仅可以扫描镜像，还能用作防毒软件。
- Snyk，漏洞检测工具，会直接链接到代码仓库，解析项目结构，并分析引入的代码及其直接和间接依赖。

运行时安全：
- Falco，孵化项目，一个云原生的运行时安全工具，检测应用、容器、主机以及 Kubernetes 的反常行为。

网络安全：
- Tigera，非开源，为Kubernetes平台提供零信任网络安全性和持续合规性解决方案。

安全审计：
- Grafeas，一个元数据和审计日志收集工具，可以用来跟踪组织中的安全合规实践。
- kube-bench，检查是否根据CIS Kubernetes基准测试中定义的安全最佳实践部署了Kubernetes。
- kube-hunter，从外部攻击者的视角在k8s集群中查找安全弱点（例如远程代码执行或者信息泄露）。
- Open Policy Agent (OPA)，孵化项目，一个开源的通用策略引擎，可以把 OPA 作为 Kubernetes 准入控制器后端进行部署，这样OPA代理可以接管安全决策，根据自定义安全约束，对请求进行校验、拒绝甚至是就地修改。
- Notary，一个允许任何人信任任意数据集合的项目，基于TUF项目，Harbor、Portieris等很多项目有集成。

基础项目：
- TUF(The Update Framework),一个针对软件分发和更新问题的安全通用设计。  

选型建议：  
目前国内私有云部署的k8s集群在安全方面关注不多，建议有较高安全需求的项目投入研究相关安全产品或选择商业端到端安全方案。

## 4.4 密钥管理
- Keycloak，用于应用程序和服务的身份和访问管理，来自Radhat。
- Athenz, 用于提供和配置（集中式授权）用例以及服务/运行时（分散式授权）用例的基于角色的授权（RBAC）系统。
- ORY Hydra，使用Go编写的OAuth2服务端框架和OpenID认证的OpenID Connect提供者。
- Vault，机密管理、加密即服务和特权访问管理的工具，可用Helm在k8s集群中部署Vault的Chart，使用Consul作为存储后端。
- Square Keywhiz，密钥分发与管理工具。
- Teleport，开源堡垒机。

本层研究较少不做选型建议。

# 5.平台
平台（Platform）包括认证发行平台（Distribution）、认证托管平台（Hosted）、认证安装工具、PaaS平台/容器服务四部分。

## 5.1 认证发行平台
![认证发行平台](/images/cncf-landscope-platform.png)
OpenShift最知名，国内平台企业占比也不少，重点推荐两个国产开源平台Kubesphere和Rancher。
- Rancher，目前用户比较多，支持多云k8s集群管理，可基于Rancher快速搭建自己的K8s集群。
- Kubesphere，我最喜欢的容器平台，界面和封装对开发友好，集成DevOps、微服务治理、灰度发布、多租户管理、工作负载和集群管理、监控告警、日志查询与收集、服务与网络、应用商店、镜像构建与镜像仓库管理和存储管理等多种业务场景。

## 5.2 认证托管平台  
![认证托管平台](/images/cncf-landscope-hosted.png)

公有云的天下，不做介绍了。

## 5.3 认证安装工具
- Kubeadm，最常用的安装工具
- Rancher Kubernetes Engine (RKE)，支持在裸机和虚拟化服务器上​​安装k8s.
- Kubesphere Installer，不在图里，但是我觉得可以加上。

## 5.4 PaaS平台/容器服务
PaaS平台Heroku、Flynn国内用的不多
- Jhipster，微服务的脚手架，快速创建Spring Boot + Angular/React项目，全栈开发人员可以关注下。

选型建议：  
本层的开源项目个人认为都不如Kubesphere实用。

彩蛋来了
- No Code，编写安全可
靠的应用程序的最佳方法：什么也不要写：什么也别部署。。。

# 6.观察与分析
观察与分析（Observability and Analysis）层包括监控、日志、追踪、混沌工程四部分。

## 6.1 监控  
![监控](/images/cncf-landscope-monitoring.png)  
- 经典组合：Prometheus（监控系统与时序数据库） + Grafana（监控、指标分析和仪表板工具），已经是很多容器平台监控标配。
- Falcon，小米开源的监控系统，国内用户不少。
- Netdata，实时Linux性能监控工具，全图Github Star数第四。

选型建议：
用业界成熟的Prometheus + Grafana组合肯定错不了；
Zabbix在传统运维监控领域仍然宝刀未老，奇怪的是Github上Star还未破千；
Beats，Elastic出品的轻量数据采集器在文件日志等场景比较有优势。

## 6.2 日志

- Fluentd，统一日志层，支持对日志进行收集、处理、转发操作。比Logstash更省资源。
- Elasticsearch，文档数据库，存储日志数据。
- Grafana Loki，日志聚合工具，类似Prometheus，但针对日志场景。

选型建议：
云原生环境下可以用Beats收集罗盘日志，使用EBK(Elasticsearch+Beats+Kibane)或者EFK组合搭建统一日志平台。

## 6.3 追踪（Tracing）
主流追踪系统有Jaeger,Pinpoint,Zipkin,CAT,Skywalking等，其中Zipkin和CAT对代码有一定的侵入性。
- Jaeger，CNCF毕业项目，来自Uber。
- OpenTracing，孵化中项目，追踪标准规范。
- Pinpoint，依赖HBase。
- Skywalking，国人开源的应用程序性能监视工具。

选型建议：
Skywalking，社区活跃，支持多种语言，案例较多。

## 6.4 混沌工程
混沌工程了解的很少，这里只介绍下阿里开源的Chaosblade。
- Chaosblade，混沌实验注入工具，可用来衡量微服务的容错能力、验证容器编排配置是否合理、测试 PaaS 层是否健壮、验证监控告警的时效性、定位与解决问题的应急能力。


# 7.Serverless  
![Serverless](/images/cncf-landscope-serverless.png)  
Serverless独立一个全景图，可见CNCF对Serverless的重视程度。
本图包括工具、安全、框架、托管平台、可安装平台等五层。

## 7.1 Serverless工具
Serverless图里面的工具开源的不多，重点介绍下下面几个：
- Hasura GraphQL Engine，基于Postgres实现细粒度访问控制的快速、实时GraphQL API，也会基于数据库事件触发webhooks。
- Node Lambda，用于在本地运行Node.js应用并将其部署到Amazon Lambda的命令行工具。

## 7.2 Serverless安全
Protego和Threat Stack都是提供Serverless安全方案的商业公司。

## 7.3 Serverless框架
Serverless框架
- Dapr，一种可移植的、事件驱动的、无服务器运行时，用于构建跨云和边缘的分布式应用程序，支持多种语言，可重点关注。
- Spring Cloud Function，提供一个通用的模型，用于在各种平台上部署基于Function的软件。
- Serverless Framework，基于Nodejs的命令行工具，使用AWS Lambda，Azure Functions，Google CloudFunctions等云产品构建无服务器架构的Web，移动和IoT应用。
- Flogo，一个基于Go的超轻量级开源生态系统，用于构建事件驱动的应用。
- Apex，使构建、部署和管理AWS Lambda功能更容易。
- AWS Serverless Application Model (SAM)，构建Serverless的框架，只能用于AWS。
- Chalice，Python Serverless框架，只能用于AWS。
- ClaudisJS，部署Node.js项目到AWS Lambda。

## 7.4 Serverless托管平台
云厂商的Serverless托管平台，不做介绍了。

## 7.5 Serverless可安装平台  
先重点介绍下Knative，基于Kubernetes的Serverless解决方案，旨在标准化Serverless，简化其学习成本。Knative包含三个主要子项目：
- Serving 提供缩容至零、请求驱动的计算功能。它本质上是无服务器平台的执行和扩展组件。
- Build 提供显式"运行至完成"功能，这对创建 CI/CD 工作流程很有用。Serving 使用它将源存储库转换为包含应用程序的容器镜像。
- Eventing 提供抽象的交付和订阅机制，允许构建松散耦合和事件驱动的无服务器应用程序。

其他Serverless可安装平台，现在是百花齐放，只介绍几个代表性的平台：
- OpenFaaS，使开发人员可以轻松地将事件驱动的函数和微服务部署到Kubernetes，而无需重复编码。可以将您的代码或现有二进制文件打包到Docker镜像。
- OpenWhisk，一个健壮的、可扩展的FaaS平台，支持数千并发触发器和调用。
- Camel K，一个轻量级集成框架，可以直接在k8s与Knative上运行Camel。  
Camel是一个基于Enterprise Integration Pattern(企业整合模式，简称EIP)的开源框架。EIP定义了一些不同应用系统之间的消息传输模型，包括常见的Point-to-Point，Pub/Sub模型。

# 8.云服务商（Special）
云服务商包括141家认证服务商和40家认证培训伙伴。


# 9.CNCF成员
CNCF成员（Members）由白金、金、银三级成员和最终用户支持者、非营利组织、学术机构组成。  
比较有意思的是不公开的机构支持者用动物图标表示。

参考资料：  
[CNCF全景图](https://landscape.cncf.io/)  
[关于 Kubernetes的33个 安全工具](https://www.jianshu.com/p/1e82c317aeb3)  
