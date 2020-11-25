---
title: "K3s快速部署到centos7"
date: 2020-06-23T15:28:30+08:00
draft: false
categories: ["k8s"]
tags: [""]
---

# 官方快速部署
1. 安装依赖
```
yum install -y container-selinux selinux-policy-base
rpm -i https://rpm.rancher.io/k3s-selinux-0.1.1-rc1.el7.noarch.rpm
```
2. 运行安装脚本
```
curl -sfL https://docs.rancher.cn/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn sh -
```

卸载
```
/usr/local/bin/k3s-uninstall.sh
```