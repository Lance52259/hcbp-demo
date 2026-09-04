# Introduction

## What is Advanced Anti-DDoS (AAD)

Advanced Anti-DDoS (AAD) is a professional DDoS protection service provided by Huawei Cloud, designed to protect internet servers and applications from distributed denial-of-service (DDoS) attacks and other malicious traffic. AAD provides comprehensive protection capabilities, including DDoS traffic scrubbing, CC (Challenge Collapsar) attack protection, and intelligent traffic analysis, ensuring the availability and stability of online services.

AAD supports multiple access modes, including website access and IP access, to meet the protection requirements of different business scenarios. By directing business traffic to AAD high-defense nodes, AAD can detect and scrub malicious traffic in real time, forwarding only legitimate traffic to the origin server, thereby effectively ensuring business continuity and security.

In addition, AAD provides black/white list management, protection policy configuration, elastic bandwidth expansion, and other functions, helping users flexibly adjust protection policies according to actual business conditions and achieve refined security operation management.

## Best Practices Overview

This section provides best practice examples for using Terraform to automatically deploy and manage Huawei Cloud Advanced Anti-DDoS (AAD), helping you understand how to efficiently manage cloud AAD resources using Infrastructure as Code (IaC).

Through the best practices in this section, you can learn the main deployment processes for AAD resources. These best practices will help you quickly get started with automated AAD deployment and lay a solid foundation for subsequent AAD management and operation work.

## Best Practices List

This section contains the following best practices:

* [Deploy Black/White Lists](black_white_lists.md) - Introduces how to use Terraform to create an AAD instance and configure black/white lists to manage access traffic.

## Reference Materials

- [Huawei Cloud Advanced Anti-DDoS Product Documentation](https://support.huaweicloud.com/aad/index.html)
- [Terraform Official Documentation](https://www.terraform.io/docs/index.html)
