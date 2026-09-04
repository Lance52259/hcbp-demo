# 简介

## 什么是DDoS高防（AAD）

DDoS高防（Advanced Anti-DDoS，AAD）是华为云提供的专业DDoS防护服务，旨在保护互联网服务器和应用免受分布式拒绝服务（DDoS）攻击及其他恶意流量的影响。AAD提供全面的防护能力，包括DDoS流量清洗、CC（Challenge Collapsar）攻击防护和智能流量分析，确保在线服务的可用性和稳定性。

AAD支持多种接入方式，包括网站接入和IP接入，能够灵活适配不同业务场景。通过将业务流量引流至AAD高防节点，AAD会对恶意流量进行实时检测和清洗，仅将合法流量转发至源站，从而有效抵御各类DDoS攻击。同时，AAD提供黑白名单、防护策略配置等功能，帮助用户精细化管理访问控制，提升业务安全性。

AAD还提供弹性带宽扩展、防护包升级等能力，用户可以根据业务需求灵活调整防护能力，实现成本与安全的平衡。此外，AAD支持企业项目管理和细粒度权限控制，便于企业统一管理和审计防护资源，满足合规要求。

## 最佳实践简述

本章节提供了使用Terraform自动化部署和管理华为云DDoS高防（AAD）的最佳实践示例，帮助您了解如何利用Infrastructure as Code（IaC）的方式高效地管理云上的AAD资源。

通过本章节的最佳实践，您可以学习到主要的AAD资源的部署流程，这些最佳实践将帮助您快速上手AAD的自动化部署，并为后续的DDoS防护管理和运维工作奠定坚实基础。

## 最佳实践列表

本章节包含以下最佳实践：

* [部署黑白名单](black_white_lists.md) - 介绍如何使用Terraform创建DDoS高防实例并配置黑白名单，实现IP级别的访问控制。

## 参考资料

- [华为云DDoS高防产品文档](https://support.huaweicloud.com/aad/index.html)
- [Terraform官方文档](https://www.terraform.io/docs/index.html)
