# AAD黑白名单最佳实践

## 应用场景

DDoS高防（Advanced Anti-DDoS，AAD）是华为云提供的专业DDoS防护服务，旨在保护互联网服务器和应用免受分布式拒绝服务（DDoS）攻击及其他恶意流量的影响。AAD提供全面的防护能力，包括DDoS流量清洗、CC（Challenge Collapsar）攻击防护和智能流量分析，确保在线服务的可用性和稳定性。

本实践将介绍如何使用Terraform创建AAD实例并配置黑白名单，以保护业务免受恶意流量的攻击。通过本实践，您可以了解AAD实例的创建流程，以及如何通过黑名单阻断恶意IP地址、通过白名单放行可信IP地址，从而提升业务的安全防护能力。

## 相关资源/数据源

本最佳实践涉及以下主要资源和数据源：

### 资源

- [DDoS高防实例（huaweicloud_aad_instance）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/aad_instance)
- [DDoS高防黑白名单（huaweicloud_aad_black_white_list）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/aad_black_white_list)

### 资源/数据源依赖关系

```
huaweicloud_aad_instance
    └── huaweicloud_aad_black_white_list
```

## 操作步骤

### 1. 脚本准备

在指定工作空间中准备好用于编写当前最佳实践脚本的TF文件（如main.tf），确保其中（也可以是其他同级目录下的TF文件）包含部署资源所需的provider版本声明和华为云鉴权信息。
配置介绍参考[部署华为云资源前的准备工作](../../introductions/prepare_before_deploy.md)一文中的介绍。

### 2. 创建AAD实例

在TF文件（如main.tf）中添加以下脚本，用于创建AAD实例。当未指定`instance_id`时，将根据`instance_config`配置创建新的AAD实例。

```hcl
# 在指定region下创建AAD实例
resource "huaweicloud_aad_instance" "test" {
  count = var.instance_id == "" ? 1 : 0

  ip_type                        = var.instance_config.ip_type
  resource_region                = var.instance_config.resource_region
  instance_access_type           = var.instance_config.instance_access_type
  duration                       = var.instance_config.duration
  amount                         = var.instance_config.amount
  instance_name                  = var.instance_config.instance_name
  period_type                    = var.instance_config.period_type
  service_bandwidth              = var.instance_config.service_bandwidth
  basic_bandwidth                = var.instance_config.basic_bandwidth
  elastic_bandwidth              = var.instance_config.elastic_bandwidth
  basic_qps                      = var.instance_config.basic_qps
  forwarding_rule                = var.instance_config.forwarding_rule
  protected_domain               = var.instance_config.protected_domain
  elastic_service_bandwidth_type = var.instance_config.elastic_service_bandwidth_type
  elastic_service_bandwidth      = var.instance_config.elastic_service_bandwidth
  protection_package             = var.instance_config.protection_package
  enterprise_project_id          = var.instance_config.enterprise_project_id

  lifecycle {
    ignore_changes = [
      ip_type,
      resource_region,
      instance_access_type,
      duration,
      amount,
      period_type,
      basic_qps,
      protection_package,
      protected_domain,
      forwarding_rule,
    ]
  }
}
```

**参数说明**：
- **ip_type**：IP类型，0表示IPv4，1表示IPv6。
- **resource_region**：资源区域，可选值包括`north_china`（华北）、`east_china`（华东）、`asia_pacific`（亚太）。
- **instance_access_type**：接入类型，0表示网站接入，1表示IP接入。
- **duration**：订购时长。
- **amount**：购买实例数量。
- **instance_name**：实例名称。
- **period_type**：订购周期类型，2表示月，3表示年。
- **service_bandwidth**：业务带宽，单位为Mbps。
- **basic_bandwidth**：基础带宽，单位为Mbps，当`resource_region`为`north_china`或`east_china`时必填，为`asia_pacific`时需为空。
- **elastic_bandwidth**：弹性带宽，单位为Mbps，当`resource_region`为`north_china`或`east_china`时必填，为`asia_pacific`时需为空。
- **basic_qps**：基础QPS，当`instance_access_type`为0（网站接入）时必填，为1（IP接入）时需为空。
- **forwarding_rule**：转发规则数，当`instance_access_type`为1（IP接入）时必填，为0（网站接入）时需为空。
- **protected_domain**：防护域名数，当`instance_access_type`为0（网站接入）时必填，为1（IP接入）时需为空。
- **elastic_service_bandwidth_type**：弹性业务带宽类型，2表示日95峰值，3表示月95峰值。
- **elastic_service_bandwidth**：弹性业务带宽增量。
- **protection_package**：防护包，可选值包括`basic`（保险防护）和`unlimited`（无限防护）。
- **enterprise_project_id**：企业项目ID。

### 3. 配置黑白名单

在TF文件（如main.tf）中添加以下脚本，用于创建黑名单和白名单。黑名单用于阻断恶意IP地址，白名单用于放行可信IP地址。

```hcl
# 添加IP地址到黑名单以阻断恶意流量
resource "huaweicloud_aad_black_white_list" "black" {
  instance_id = var.instance_id != "" ? var.instance_id : try(huaweicloud_aad_instance.test[0].id, "")
  type        = "black"
  ips         = var.blacklist_ips
}

# 添加IP地址到白名单以放行可信流量
resource "huaweicloud_aad_black_white_list" "white" {
  instance_id = var.instance_id != "" ? var.instance_id : try(huaweicloud_aad_instance.test[0].id, "")
  type        = "white"
  ips         = var.whitelist_ips
}
```

**参数说明**：
- **instance_id**：AAD实例ID。当指定`instance_id`时，使用已有实例；否则使用新创建的实例ID。
- **type**：名单类型，`black`表示黑名单，`white`表示白名单。
- **ips**：IP地址列表。

### 4. 预设资源部署所需的入参

本实践中，部分资源、数据源使用了输入变量对配置内容进行赋值，这些输入参数在后续部署时需要手工输入。
同时，Terraform提供了通过`tfvars`文件预设这些配置的方法，可以避免每次执行时重复输入。

在工作目录下创建`terraform.tfvars`文件，示例内容如下：

```hcl
# AAD实例配置
instance_config = {
  instance_name        = "aad-instance"
  ip_type              = 0
  resource_region      = "north_china"
  instance_access_type = "1"
  duration             = 1
  amount               = 1
  period_type          = 2
  service_bandwidth    = 10
  basic_bandwidth      = 10
  elastic_bandwidth    = 10
  forwarding_rule      = 50
}

# 黑白名单
blacklist_ips = ["192.170.1.10", "192.170.1.11"]
whitelist_ips = ["192.170.1.200", "192.170.1.201"]
```

**使用方法**：

1. 将上述内容保存为工作目录下的`terraform.tfvars`文件（该文件名可使用户在执行terraform命令时自动导入该`tfvars`文件中的内容，其他命名则需要在tfvars前补充`.auto`定义，如`variables.auto.tfvars`）
2. 根据实际需要修改参数值
3. 执行`terraform plan`或`terraform apply`时，Terraform会自动读取该文件中的变量值

除了使用`terraform.tfvars`文件外，还可以通过以下方式设置变量值：

1. 命令行参数：`terraform apply -var="instance_name=my-aad-instance"`
2. 环境变量：`export TF_VAR_instance_name=my-aad-instance`
3. 自定义命名的变量文件：`terraform apply -var-file="custom.tfvars"`

> 注意：如果同一个变量通过多种方式进行设置，Terraform会按照以下优先级使用变量值：命令行参数 > 变量文件 > 环境变量 > 默认值。

### 5. 初始化并应用Terraform配置

完成以上脚本配置后，执行以下步骤来创建资源：

1. 运行 `terraform init` 初始化环境
2. 运行 `terraform plan` 查看资源创建计划
3. 确认资源计划无误后，运行 `terraform apply` 开始创建AAD实例及黑白名单
4. 运行 `terraform show` 查看已创建的AAD实例及黑白名单

## 参考信息

- [华为云DDoS高防产品文档](https://support.huaweicloud.com/aad/index.html)
- [华为云Provider文档](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs)
- [AAD黑白名单最佳实践源码参考](https://github.com/huaweicloud/terraform-provider-huaweicloud/tree/master/examples/aad/black-white-lists)
