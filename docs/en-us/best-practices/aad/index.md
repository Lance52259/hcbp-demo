# Introduction

## What is Advanced Anti-DDoS (AAD)

Advanced Anti-DDoS (AAD) is a professional DDoS protection service provided by Huawei Cloud, designed to protect internet servers and applications from distributed denial-of-service (DDoS) attacks and other malicious traffic. AAD provides comprehensive protection capabilities, including DDoS traffic scrubbing, CC (Challenge Collapsar) attack protection, and intelligent traffic analysis, ensuring the availability and stability of online services.

AAD supports multiple access modes, including website access and IP access, and can flexibly adapt to different business scenarios. By directing business traffic to AAD high-defense nodes, AAD performs real-time detection and scrubbing of malicious traffic, forwarding only legitimate traffic to the origin server, thereby effectively defending against various DDoS attacks. Additionally, AAD provides functions such as black/white lists and protection policy configuration, helping users manage access control in a refined manner and enhance business security.

AAD also offers elastic bandwidth expansion, protection package upgrades, and other capabilities, allowing users to flexibly adjust protection capabilities based on business needs, achieving a balance between cost and security. Furthermore, AAD supports enterprise project management and fine-grained permission control, facilitating unified management and auditing of protection resources and meeting compliance requirements.

## Best Practices Overview

This section provides best practice examples for using Terraform to automatically deploy and manage Huawei Cloud Advanced Anti-DDoS (AAD), helping you understand how to efficiently manage cloud AAD resources using Infrastructure as Code (IaC).

Through the best practices in this section, you can learn the main deployment processes for AAD resources. These best practices will help you quickly get started with automated AAD deployment and lay a solid foundation for subsequent DDoS protection management and operation work.

## Best Practices List

This section contains the following best practices:

* [Deploy Black/White Lists](black_white_lists.md) - Introduces how to use Terraform to create an AAD instance and configure black/white lists for IP-level access control.

## Reference Materials

- [Huawei Cloud Advanced Anti-DDoS Product Documentation](https://support.huaweicloud.com/aad/index.html)
- [Terraform Official Documentation](https://www.terraform.io/docs/index.html)
