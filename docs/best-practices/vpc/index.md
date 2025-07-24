# 虚拟私有云（VPC）最佳实践

## 概述

虚拟私有云（Virtual Private Cloud，VPC）是华为云为用户提供的逻辑隔离网络环境，支持自定义子网、路由、ACL等网络资源，满足企业级网络安全和灵活组网需求。

本章节提供了使用Terraform自动化部署和管理华为云VPC的最佳实践示例，帮助您了解如何利用Infrastructure as Code（IaC）的方式高效地管理云上的网络资源。

## 最佳实践列表

本章节包含以下最佳实践：

* [基础VPC网络部署](basic.md) - 介绍如何使用Terraform自动化部署基础VPC及子网。
* [VPC Peering连接](peering.md) - 介绍如何使用Terraform自动化部署两个VPC及其子网，并建立VPC Peering连接和路由。

## 参考资料

- [华为云VPC产品文档](https://support.huaweicloud.com/vpc/index.html)
- [Terraform官方文档](https://www.terraform.io/docs/index.html)
