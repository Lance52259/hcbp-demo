# Web应用防火墙（WAF）最佳实践

## 产品介绍

Web应用防火墙（Web Application Firewall，简称WAF）是华为云为Web应用提供的一站式安全防护服务，
可有效防御SQL注入、XSS跨站脚本、网页木马上传、命令注入、恶意爬虫、CC攻击等多种Web安全威胁。
支持云模式和独享实例（专业版），满足企业级高性能和合规需求。

### 主要特性

- **多维度Web攻击防护**：防御SQL注入、XSS、命令注入、文件上传等主流Web攻击
- **灵活部署**：支持云模式和独享实例，满足不同业务场景
- **智能防护引擎**：持续更新规则库，具备自学习能力
- **高性能与弹性**：独享资源，性能保障，支持弹性扩展
- **可视化运维**：实时监控、日志分析、策略灵活配置

## 最佳实践目录

1. [使用Terraform部署WAF专业版实例](./dedicated_instance.md)
   - 通过Terraform实现自动化部署
   - 支持独享实例的资源创建
   - 提供完整的网络和安全配置

## 使用场景

1. **网站安全防护**
   - 防止SQL注入、XSS等攻击
   - 满足等保合规
   - 保护网站可用性

2. **电商/金融/政企门户**
   - 保护交易和敏感数据
   - 防止恶意爬虫和CC攻击
   - 满足监管要求

3. **API服务防护**
   - 保护API接口安全
   - 防止恶意调用
   - 控制访问频率

## 产品优势

1. **全面防护**
   - 多种攻击检测与拦截
   - 持续更新的防护规则
2. **高性能与弹性**
   - 独享资源，性能保障
   - 支持弹性扩展
3. **安全合规**
   - 满足主流合规要求
   - 数据隔离与访问控制
4. **易用性**
   - 可视化管理
   - 自动化部署与运维

## 参考文档

- [WAF产品文档](https://support.huaweicloud.com/waf/index.html)
- [WAF最佳实践](https://support.huaweicloud.com/bestpractice-waf/waf_06_0001.html)
- [Terraform部署指南](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/waf_dedicated_instance) 