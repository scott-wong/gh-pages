---
title: "企业架构系列：开发规范"
date: 2020-04-01T09:38:36+08:00
draft: false
categories: ["架构"]
tags: ["开发规范"]
---
# 企业架构系列：开发规范

## 版本号
【强制】版本号遵循 [SemVer 2 标准](https://semver.org/)  

**语义化版本 2.0.0**  
版本格式：主版本号.次版本号.修订号，版本号递增规则如下：

- 主版本号：当你做了不兼容的 API 修改，
- 次版本号：当你做了向下兼容的功能性新增，
- 修订号：当你做了向下兼容的问题修正。  

先行版本号及版本编译元数据可以加到“主版本号.次版本号.修订号”的后面，作为延伸。

## 文档格式
【推荐】Markdown格式  
tips: 
md转word  
- 使用VSCode的Markdown预览，直接复制到Word里，保持格式。  
- [writage插件](http://www.writage.com)


## 架构设计规范
### TOGAF

## 后端开发规范
### Java
【强制】[《Java开发手册》](https://developer.aliyun.com/special/tech-java),当前最新版1.5.0  
【强制】IDEA插件[Alibaba Java Coding Guidelines](https://plugins.jetbrains.com/plugin/10046-alibaba-java-coding-guidelines)

## 前端开发规范
### Vue
[Vue前端开发规范](https://www.jianshu.com/p/8b095857f73e)

## 数据传输规范
### 日期时间字段
使用ISO 8601-1：2019规范中的扩展格式，如日期时间组合的格式为
```
2020-04-01T09:38:36+08:00
```