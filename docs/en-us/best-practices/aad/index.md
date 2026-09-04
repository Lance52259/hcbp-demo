# Introduction

## What is Advanced Anti-DDoS (AAD)

Advanced Anti-DDoS (AAD) is a professional DDoS protection service provided by Huawei Cloud, designed to protect internet servers and applications from distributed denial-of-service (DDoS) attacks and other malicious traffic. AAD provides comprehensive protection capabilities, including DDoS traffic cleaning, CC (Challenge Collapsar) attack protection, and intelligent traffic analysis, ensuring the availability and stability of online services.

AAD supports multiple access modes, including website access and IP access, allowing flexible selection based on business requirements. Meanwhile, AAD provides various protection policies, such as basic protection, elastic protection, and unlimited protection, helping users choose according to actual attack scale and budget. In addition, AAD supports black/white list configuration, allowing users to add malicious IP addresses to the blacklist for blocking and trusted IP addresses to the whitelist for allowing, thereby achieving fine-grained management of access traffic.

AAD also provides detailed attack logs and monitoring reports to help users understand the attack situation and protection effect in real time. By collaborating with other Huawei Cloud security services (such as Cloud Firewall and Web Application Firewall), AAD can build a multi-layered defense-in-depth system to comprehensively enhance the security protection capability of businesses.

## Best Practices Overview

This section provides best practice examples for using Terraform to automatically deploy and manage Huawei Cloud Advanced Anti-DDoS (AAD), helping you understand how to efficiently manage cloud AAD resources using Infrastructure as Code (IaC).

Through the best practices in this section, you can learn the main deployment processes for AAD resources. These best practices will help you quickly get started with automated AAD deployment and lay a solid foundation for subsequent AAD management and operation work.

## Best Practices List

This section contains the following best practices:

* [Deploy Black/White Lists](black_white_lists.md) - Introduces how to use Terraform to create an AAD instance and configure black/white lists to enhance the security protection of your business.

## Reference Materials

- [Huawei Cloud Advanced Anti-DDoS Product Documentation](https://support.huaweicloud.com/aad/index.html)
- [Terraform Official Documentation](https://www.terraform.io/docs/index.html)
