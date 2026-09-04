# 简介

## 什么是DDoS高防（AAD）

DDoS高防（Advanced Anti-DDoS，AAD）是华为云提供的专业DDoS防护服务，旨在保护互联网服务器和应用免受分布式拒绝服务（DDoS）攻击及其他恶意流量的影响。AAD提供全面的防护能力，包括DDoS流量清洗、CC（Challenge Collapsar）攻击防护和智能流量分析，确保在线服务的可用性和稳定性。

AAD支持多种接入方式，包括网站接入和IP接入，并提供灵活的带宽配置，以满足不同业务场景的防护需求。同时，AAD提供黑白名单管理功能，您可以阻断恶意IP地址并放行可信IP地址，从而提升业务的安全防护能力。

AAD可与华为云其他服务集成，例如云监控服务（CES）用于监控、消息通知服务（SMN）用于告警通知，帮助您构建全面的安全防护体系。通过AAD，您可以有效防御DDoS攻击，确保在线服务的持续可用。

## 最佳实践简述

本章节提供了使用Terraform自动化部署和管理华为云DDoS高防（AAD）的最佳实践示例，帮助您了解如何利用Infrastructure as Code（IaC）的方式高效地管理云上的AAD资源。

通过本章节的最佳实践，您可以学习到主要的AAD资源的部署流程，这些最佳实践将帮助您快速上手AAD的自动化部署，并为后续的AAD管理和运维工作奠定坚实基础。

## 最佳实践列表

本章节包含以下最佳实践：

* [AAD黑白名单最佳实践](black_white_lists.md) - 介绍如何使用Terraform创建AAD实例并配置黑白名单，以保护业务免受恶意流量的攻击。

## 参考资料

- [华为云DDoS高防产品文档](https://support.huaweicloud.com/aad/index.html)
- [Terraform官方文档](https://www.terraform.io/docs/index.html)
