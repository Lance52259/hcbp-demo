# Introduction

## What is Distributed Cache Service (DCS)

Distributed Cache Service (DCS) is a high-performance, highly available in-memory database service provided by Huawei Cloud, supporting mainstream cache engines such as Redis and Memcached. DCS service provides multiple instance specifications and deployment modes, including single-node, master-standby, and cluster, meeting cache requirements for different scales and scenarios.

DCS service provides complete cache lifecycle management functionality, supporting enterprise-level features such as automatic backup, monitoring alerts, and parameter tuning, with high reliability and security. Through DCS service, enterprises can easily implement application scenarios such as data caching, session storage, and message queues, improving application performance and user experience.

## Best Practices Overview

This section provides best practice examples for using Terraform to automatically deploy and manage Huawei Cloud Distributed Cache Service (DCS), helping you understand how to efficiently manage cloud cache resources using Infrastructure as Code (IaC).

Through the best practices in this section, you can learn the main deployment processes for DCS resources. These best practices will help you quickly get started with automated DCS deployment and lay a solid foundation for subsequent cache management and operation work.

## Best Practices List

This section contains the following best practices:

* [Deploy Single-Node Redis Instance](redis_single_instance.md) - Introduces how to use Terraform to automatically deploy DCS single-node Redis instances, including VPC creation, instance configuration, and basic network setup.
* [Deploy Master-Standby Redis Instance](redis_ha_instance.md) - Introduces how to use Terraform to automatically deploy DCS master-standby Redis instances, including VPC creation, instance configuration, backup policy, and whitelist management.

## Reference Materials

- [Huawei Cloud DCS Product Documentation](https://support.huaweicloud.com/dcs/index.html)
- [Terraform Official Documentation](https://www.terraform.io/docs/index.html)
