# 部署黑白名单

## 应用场景

DDoS高防（Advanced Anti-DDoS，AAD）是华为云提供的专业DDoS防护服务，旨在保护互联网服务器和应用免受分布式拒绝服务（DDoS）攻击及其他恶意流量的影响。AAD提供全面的防护能力，包括DDoS流量清洗、CC（Challenge Collapsar）攻击防护和智能流量分析，确保在线服务的可用性和稳定性。

本最佳实践将介绍如何使用Terraform创建AAD实例并配置黑白名单，以帮助您快速实现对恶意IP地址的封禁和可信IP地址的放行，提升业务的安全防护能力。

## 相关资源/数据源

本最佳实践涉及以下主要资源：

### 资源

- [DDoS高防实例（huaweicloud_aad_instance）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/aad_instance)
- [DDoS高防黑白名单（huaweicloud_aad_black_white_list）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/aad_black_white_list)

### 资源/数据源依赖关系

```
huaweicloud_aad_instance
    ├── huaweicloud_aad_black_white_list.black
    └── huaweicloud_aad_black_white_list.white
```

## 操作步骤

### 1. 脚本准备

在指定工作空间中准备好用于编写当前最佳实践脚本的TF文件（如main.tf），确保其中（也可以是其他同级目录下的TF文件）包含部署资源所需的provider版本声明和华为云鉴权信息。
配置介绍参考[部署华为云资源前的准备工作](../../introductions/prepare_before_deploy.md)一文中的介绍。

### 2. 创建DDoS高防实例

在TF文件（如main.tf）中添加以下脚本，用于在未指定已有实例ID时创建新的AAD实例：

```hcl
# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下创建DDoS高防实例
variable "instance_config" {
  description = "The configuration for creating a new AAD instance"

  type = object({
    instance_name                  = string
    ip_type                        = number
    resource_region                = string
    instance_access_type           = string
    duration                       = number
    amount                         = number
    period_type                    = number
    service_bandwidth              = number
    basic_bandwidth                = optional(number, null)
    elastic_bandwidth              = optional(number, null)
    basic_qps                      = optional(number, null)
    forwarding_rule                = optional(number, null)
    protected_domain               = optional(number, null)
    elastic_service_bandwidth_type = optional(number, null)
    elastic_service_bandwidth      = optional(number, null)
    protection_package             = optional(string, null)
    enterprise_project_id          = optional(string, null)
  })

  default = null
}

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
- **count**：通过判断变量 instance_id 是否为空来决定是否创建新实例，当 instance_id 为空时创建，否则不创建。
- **ip_type**：通过引用输入变量 instance_config 中的 ip_type 进行赋值，用于指定IP类型。
- **resource_region**：通过引用输入变量 instance_config 中的 resource_region 进行赋值，用于指定资源区域。
- **instance_access_type**：通过引用输入变量 instance_config 中的 instance_access_type 进行赋值，用于指定接入类型。
- **duration**：通过引用输入变量 instance_config 中的 duration 进行赋值，用于指定订购时长。
- **amount**：通过引用输入变量 instance_config 中的 amount 进行赋值，用于指定购买数量。
- **instance_name**：通过引用输入变量 instance_config 中的 instance_name 进行赋值，用于指定实例名称。
- **period_type**：通过引用输入变量 instance_config 中的 period_type 进行赋值，用于指定订购周期类型。
- **service_bandwidth**：通过引用输入变量 instance_config 中的 service_bandwidth 进行赋值，用于指定业务带宽。
- **basic_bandwidth**：通过引用输入变量 instance_config 中的 basic_bandwidth 进行赋值，用于指定基础带宽。
- **elastic_bandwidth**：通过引用输入变量 instance_config 中的 elastic_bandwidth 进行赋值，用于指定弹性带宽。
- **basic_qps**：通过引用输入变量 instance_config 中的 basic_qps 进行赋值，用于指定基础QPS。
- **forwarding_rule**：通过引用输入变量 instance_config 中的 forwarding_rule 进行赋值，用于指定转发规则数。
- **protected_domain**：通过引用输入变量 instance_config 中的 protected_domain 进行赋值，用于指定防护域名数。
- **elastic_service_bandwidth_type**：通过引用输入变量 instance_config 中的 elastic_service_bandwidth_type 进行赋值，用于指定弹性业务带宽类型。
- **elastic_service_bandwidth**：通过引用输入变量 instance_config 中的 elastic_service_bandwidth 进行赋值，用于指定弹性业务带宽增量。
- **protection_package**：通过引用输入变量 instance_config 中的 protection_package 进行赋值，用于指定防护包。
- **enterprise_project_id**：通过引用输入变量 instance_config 中的 enterprise_project_id 进行赋值，用于指定企业项目ID。

### 3. 添加黑名单

在TF文件（如main.tf）中添加以下脚本，用于将恶意IP地址加入黑名单：

```hcl
# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下添加黑名单
variable "blacklist_ips" {
  description = "The list of IP addresses to add to the blacklist"
  type        = list(string)
}

resource "huaweicloud_aad_black_white_list" "black" {
  instance_id = var.instance_id != "" ? var.instance_id : try(huaweicloud_aad_instance.test[0].id, "")
  type        = "black"
  ips         = var.blacklist_ips
}
```

**参数说明**：
- **instance_id**：通过判断变量 instance_id 是否为空来选择使用已有实例ID或新创建实例的ID。
- **type**：固定为"black"，表示黑名单。
- **ips**：通过引用输入变量 blacklist_ips 进行赋值，用于指定需要加入黑名单的IP地址列表。

### 4. 添加白名单

在TF文件（如main.tf）中添加以下脚本，用于将可信IP地址加入白名单：

```hcl
# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下添加白名单
variable "whitelist_ips" {
  description = "The list of IP addresses to add to the whitelist"
  type        = list(string)
}

resource "huaweicloud_aad_black_white_list" "white" {
  instance_id = var.instance_id != "" ? var.instance_id : try(huaweicloud_aad_instance.test[0].id, "")
  type        = "white"
  ips         = var.whitelist_ips
}
```

**参数说明**：
- **instance_id**：通过判断变量 instance_id 是否为空来选择使用已有实例ID或新创建实例的ID。
- **type**：固定为"white"，表示白名单。
- **ips**：通过引用输入变量 whitelist_ips 进行赋值，用于指定需要加入白名单的IP地址列表。

### 5. 预设资源部署所需的入参（可选）

本实践中，部分资源、数据源使用了输入变量对配置内容进行赋值，这些输入参数在后续部署时需要手工输入。
同时，Terraform提供了通过`tfvars`文件预设这些配置的方法，可以避免每次执行时重复输入。

在工作目录下创建`terraform.tfvars`文件，示例内容如下：

```hcl
# 根据脚本变量填写；敏感信息使用占位符
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

### 6. 初始化并应用Terraform配置

完成以上脚本配置后，执行以下步骤来创建资源：

1. 运行 `terraform init` 初始化环境
2. 运行 `terraform plan` 查看资源创建计划
3. 确认资源计划无误后，运行 `terraform apply` 开始创建DDoS高防实例及黑白名单
4. 运行 `terraform show` 查看已创建的DDoS高防实例及黑白名单

## 参考信息

- [华为云DDoS高防产品文档](https://support.huaweicloud.com/aad/index.html)
- [华为云Provider文档](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs)
- [AAD黑白名单最佳实践源码参考](https://github.com/huaweicloud/terraform-provider-huaweicloud/tree/master/examples/aad/black-white-lists)
