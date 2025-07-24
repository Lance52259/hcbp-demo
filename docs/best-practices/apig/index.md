# API网关（APIG）最佳实践

## 概述

API网关（API Gateway）是为企业和开发者提供的高性能、高可用、高安全的云原生网关服务。它能快速将企业服务能力包装成标准API接口，帮助您轻松构建、管理和部署任意规模的API，并支持API上架云商店进行售卖。借助API网关，您可以简单、快速、低成本、低风险地实现内部系统集成和业务能力开放。

API网关提供完整的API生命周期管理功能，支持多种安全认证和防护机制，具备云原生网关能力，可简化架构并降低部署和运维成本。通过API网关，企业可以快速实现业务能力开放，构建API生态，实现业务价值最大化。

本章节提供了使用Terraform自动化部署和管理华为云API网关（APIG）的最佳实践示例，帮助您了解如何利用Infrastructure as Code（IaC）的方式高效地管理云上的API网关资源。

## 最佳实践列表

本章节包含以下最佳实践：

* [使用FunctionGraph实现APIG自定义认证](function_authorizer.md) - 介绍如何使用Terraform自动化部署API网关自定义认证，以FunctionGraph函数实现认证逻辑。

## 参考资料

- [华为云API网关产品文档](https://support.huaweicloud.com/apig/index.html)
- [Terraform官方文档](https://www.terraform.io/docs/index.html)
