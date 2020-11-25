---
title: "企业架构系列：云原生基础设施选型"
date: 2020-03-25T09:56:21+08:00
draft: false
categories: ["架构"]
tags: ["云原生", "TOGAF","C4模型", "敏捷"]
---

# 企业架构系列：云原生基础设施选型
为了降低企业IT综合成本，企业全面上云已是大势所趋，但是上云路径多种多样，尽管公有云优势明显，但因监管需要、数据安全等因素，大公司很多仍倾向自建私有云或混合云，不希望被单一公有云捆绑。  
近年来云原生架构日趋成熟，常用的k8s+docker+devops工具即可搭建，但是在私有云上自建维护一套这样的云原生基础设施绝非易事，通常需要成立独立的团队维护。  
本文基于作者多年跨行业项目实践经验和数月来的研究提出云原生基础设施的选型建议，主要面向有私有云部署需求的公司，小公司建议直接上公有云。  
毕竟每家公司的组织结构和业务技术需求千差万别，文中的选型也没法做到面面俱到，如有疏漏和错误，敬请指正。

---

## 什么是云原生
CNCF云原生定义：  
云原生技术有利于各组织在公有云、私有云和混合云等新型动态环境中，构建和运行可弹性扩展的应用。云原生的代表技术包括容器、服务网格、微服务、不可变基础设施和声明式 API。  
![云原生定义](/images/cloudnative.png)

## 云原生生态图
从全局视角了解云原生生态可以直接看[CNCF全景图](https://landscape.cncf.io/)，截止2020年4月27日已有1,382个图标。包含应用定义与开发层（App Definition and Development）、编排与治理层（Orchestration Management）、运行时层（Runtime）、供应保障层（Provisioning）、云服务商（Special）、平台（Platform）、观察与分析（Observability and Analysis）、无服务（Serverless）和CNCF成员（Members）八个层次。  
![CNCF全景图](/images/cncf-landscope.png)
作者并没有全面研究每个层级，下面仅列出日常用到的产品或工具，下篇文章计划按全景图的层级划分重新梳理选型。  


## 1. 选型清单
- 企业架构设计：TOGAF + [C4模型](https://c4model.com/)
- 敏捷项目管理工具&知识共享：[TAPD](https://www.tapd.cn/)
- 容器管理平台：[Kubesphere](https://kubesphere.com.cn/)
- 镜像仓库&镜像安全：Harbor + Clair
- 代码管理工具&代码评审：Gitlab
- CI/CD工具：Kubesphere集成的Jenkins流水线
- 代码质量扫描工具：Kubesphere集成的SonarQube
- 应用定义工具：Kubesphere集成的Helm
- APM&调用链追踪：SkyWalking

## 2. 具体选型逻辑
### 2.1 企业架构设计：TOGAF + [C4模型](https://c4model.com/)
**TOGAF**  
关于企业架构（EA）设计业界主要有美国联邦政府体系架构框架FEA(F)、美国国防部体系架构框架（DoDAF）和开放组织体系架构框架（TOGAF），目前各个框架在发展过程中都在互相借鉴。针对企业市场TOGAF更加适合而且适合按需裁剪。另外电子政务从业者建议看下《电子政务顶层设计理论、方法与实践》这本书，该书作者结合TOGAF等框架和中国电子政务特色提出了CGAF框架。
TOGAF的核心是架构开发方法ADM，计划独立成文描述TOGAF在敏捷项目交付的实践。  
![ADM开发方法](/images/togaf-adm.png)   
**C4模型**  
工作中经常会与业务和技术人员沟通架构设计，发现每个人画出来的架构图都不同，尤其在敏捷开发盛行的当下，架构文档在大幅缩减，很多人已经弃用UML。最近发现C4模型能很好的描述软件架构，而且容易上手易于理解。 
C4模型是Simon Brown创建，目前欧美用的比较多，国内相关使用案例找到的不多。但是作者认为可以在项目团队中广泛使用。  
C4 代表上下文（Context）、容器（Container）、组件（Component）和代码（Code）——一系列分层的图表，可以用这些图表来描述不同缩放级别的软件架构，每种图表都适用于不同的受众。可以将其视为代码的地图。
C4包含四种核心图和三种附加图：
- 系统上下文图(System Context diagram)
- 容器图(Container diagram)
- 组件图(Component diagram)
- 代码图(Code diagram), 使用UML类图、实体关系图等，通常不需要画，IDE生成即可
- 系统全景图(System Landscape diagram)
- 动态图(Dynamic diagram), 类似UML通讯图，可让初级开发人员在写代码前画出来评估，避免后期才发现错误
- 部署图(Deployment diagram), 可用于开发与运维沟通
详细内容可访问C4模型官网，介绍的很清楚，具体画图可用我基于RicardoNiepel/C4-PlantUML优化的[C4-PlantUML](https://github.com/scott-wong/C4-PlantUML)或者使用draw.io+[C4插件](https://tobiashochguertel.github.io/c4-draw.io/c4.js).  
![C4模型概览](/images/c4-overview.png)

### 2.2 敏捷项目管理工具：[TAPD](https://www.tapd.cn/)
备选：有独立部署需求选[禅道](https://www.zentao.net/)  
TAPD的优势
- 提供轻量协作、敏捷研发完整支持
- 跨端支持，集成企业微信
- 文档和Wiki可用于知识共享
企业版有DevOps功能，这个禅道开源版有提供，可按需选择。

### 2.3 容器管理平台[Kubesphere](https://kubesphere.com.cn/)
备选：Rancher  
Kubesphere和Rancher都是开源产品，Rancher相对成熟些，案例比较多，不过Kubesphere相比Rancher提供了针对开发人员更友好的界面操作并深度集成Jenkins、SonarQube等CI/CD工具，同时提供统一日志、统一监控告警等功能。
当前版本2.1.1已生产可用，据说光大银行在测试Kubesphere。
Kubesphere优势
- 通过Helm快速部署mysql、redis等应用
- 集成应用商店openpitrix，可上传自有应用
- 统一日志、统一监控告警

### 2.4 代码管理工具&代码评审：Gitlab
备选：Gitee, Github  
代码管理没多少选择，自建Gitlab开源版提供的功能够用了。
代码审查还有Gerrit可选，不过Gitlab也能通过merge request进行代码评审，而且更符合开发人员习惯。

### 2.5 CI/CD工具：Kubesphere集成的Jenkins+SonarQube流水线
在代码目录配置Jenkinsfile，通过Kubesphere实现自动拉取代码并构建maven, nodejs等项目工程，代码质量扫描，自动构建成Docker镜像并发布到指定k8s集群中。
典型Java项目的Jenkinsfile如下：
```
pipeline {
    agent {
        node {
            label 'maven'
        }
    }

    parameters {
        string(name:'TAG_NAME',defaultValue: '',description:'')
    }

    environment {
        // app label
        PROJECT = 'ry'
        NAMESPACE = 'cargo-dev'
        // 服务唯一标识
        APP_NAME = 'ruoyi-api'
        // 项目git地址，用于push with tag
        GIT_URL = 'gitee.com/scott-wong/RuoYi-Vue.git'

        // 以下是devops公共配置
        // 镜像密钥，官方dockerhub-cr，阿里云aliyun-cr
        DOCKER_SECRETS = 'aliyun-cr'
        GIT_CREDENTIAL_ID = 'gitee-id'
        KUBECONFIG_CREDENTIAL_ID = 'cargo-kubeconfig'
        // 阿里云镜像用aliyun-id
        DOCKER_CREDENTIAL_ID = 'aliyun-id'
        // 阿里云registry.cn-hangzhou.aliyuncs.com
        REGISTRY = 'registry.cn-hangzhou.aliyuncs.com'
        // 阿里云scottwong
        DOCKERHUB_NAMESPACE = 'scottwong'
        // 代码检测
        SONAR_CREDENTIAL_ID= 'sonar-token'
    }

    stages {
        stage ('checkout scm') {
            steps {
                checkout(scm)
            }
        }

        stage ('unit test') {
            steps {
                container ('maven') {
                    sh 'cd ruoyi && mvn clean test'
                }
            }
        }

        stage('sonarqube analysis') {
            steps {
                container ('maven') {
                    withCredentials([string(credentialsId: "$SONAR_CREDENTIAL_ID", variable: 'SONAR_TOKEN')]) {
                        withSonarQubeEnv('sonar') {
                            sh "cd ruoyi && mvn sonar:sonar -Dsonar.java.binaries=target/classes -Dsonar.branch=$BRANCH_NAME -Dsonar.login=$SONAR_TOKEN"
                        }
                    }
                    timeout(time: 1, unit: 'HOURS') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }

        stage ('build & push') {
            steps {
                container ('maven') {
                    sh 'cd ruoyi && mvn -Dmaven.test.skip=true clean package'
                    sh 'cd ruoyi && docker build -f Dockerfile -t $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER .'
                    withCredentials([usernamePassword(passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,credentialsId : "$DOCKER_CREDENTIAL_ID" ,)]) {
                        sh 'echo "$DOCKER_PASSWORD" | docker login $REGISTRY -u "$DOCKER_USERNAME" --password-stdin'
                        sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER'
                    }
                }
            }
        }

        stage('push latest'){
           when{
                branch 'master'
           }
           steps{
                container ('maven') {
                  sh 'docker tag  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:latest '
                  sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:latest '
                }
           }
        }

        stage('deploy to dev') {
            when{
                branch 'master'
            }
            steps {
                input(id: 'deploy-to-dev', message: 'deploy to dev?')
                kubernetesDeploy(configs: 'ruoyi/deploy/dev/**', enableConfigSubstitution: true, kubeconfigId: "$KUBECONFIG_CREDENTIAL_ID")
            }
        }

        stage('push with tag'){
            when{
                expression{
                    return params.TAG_NAME =~ /v.*/
                }
            }
            steps {
                container ('maven') {
                    input(id: 'release-image-with-tag', message: 'release image with tag?')
                    withCredentials([usernamePassword(credentialsId: "$GIT_CREDENTIAL_ID", passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh 'git config --global user.email "xxx" '
                        sh 'git config --global user.name "scott-wong" '
                        sh 'git tag -a $TAG_NAME -m "$TAG_NAME" '
                        sh 'git push https://$GIT_USERNAME:$GIT_PASSWORD@$GIT_URL --tags --ipv4'
                    }
                    sh 'docker tag  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:$TAG_NAME '
                    sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:$TAG_NAME '
                }
            }
        }
    }
}
```

## 2.6 应用定义工具：Helm
Helm几乎是目前的事实标准，另一种应用定义模型OAM也值得关注。
阿里巴巴宣布联合微软共同推出的开放应用模型项目（Open Application Model - OAM）


 