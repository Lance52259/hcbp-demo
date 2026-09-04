# 简介

## 什么是DDoS高防（AAD）

DDoS高防（Advanced Anti-DDoS，AAD）是华为云提供的专业DDoS防护服务，旨在保护互联网服务器和应用免受分布式拒绝服务（DDoS）攻击及其他恶意流量的影响。AAD提供全面的防护能力，包括DDoS流量清洗、CC（Challenge Collapsar）攻击防护和智能流量分析，确保在线服务的可用性和稳定性。

AAD提供灵活的部署模式以满足不同的业务需求。您可以选择网站接入或IP接入模式，并可从华北、华东、亚太等多个资源区域中进行选择。该服务同时支持IPv4和IPv6，并提供包括基础带宽、弹性带宽和业务带宽在内的多种带宽选项，以适应不同的流量规模。此外，AAD与企业项目管理集成，允许您将实例关联到特定的企业项目，以实现更好的资源治理。

借助AAD，您可以有效缓解DDoS攻击的影响，保持业务连续性，并保护品牌声誉。该服务提供实时攻击监控和告警，使您能够快速响应安全威胁。通过利用AAD的黑白名单功能，您可以精确控制允许或阻止的IP地址，进一步增强安全态势。

## 最佳实践简述

本章节提供了使用Terraform自动化部署和管理华为云DDoS高防（AAD）的最佳实践示例，帮助您了解如何利用Infrastructure as Code（IaC）的方式高效地管理云上的AAD资源。

通过本章节的最佳实践，您可以学习到主要的AAD资源的部署流程，这些最佳实践将帮助您快速上手AAD的自动化部署，并为后续的AAD管理和运维工作奠定坚实基础。

## 最佳实践列表

本章节包含以下最佳实践：

* [AAD黑白名单](black_white_lists.md) - 介绍如何使用Terraform创建AAD实例并配置黑白名单，以封禁恶意IP地址并放行可信IP地址。

## 参考资料

- [华为云DDoS高防产品文档](https://support.huaweicloud.com/aad/index.html)
- [Terraform官方文档](https://www.terraform.io/docs/index.html)
