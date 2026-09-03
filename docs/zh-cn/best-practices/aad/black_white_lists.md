# 部署AAD黑白名单

## 应用场景

华为云AAD（Anti-DDoS）服务为公网IP提供DDoS攻击防护。本实践将指导您使用Terraform创建AAD实例，并为其配置IP黑白名单，以阻断恶意流量并放行可信流量，保障业务稳定运行。

## 相关资源/数据源

本最佳实践涉及以下主要资源和数据源：

### 数据源

本实践未使用数据源。

### 资源

- [AAD实例（huaweicloud_aad_instance）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/aad_instance)
- [AAD黑白名单（huaweicloud_aad_black_white_list）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/aad_black_white_list)

### 资源/数据源依赖关系

```
huaweicloud_aad_instance
    └── huaweicloud_aad_black_white_list
```

## 操作步骤

### 1. 脚本准备

在指定工作空间中准备好用于编写当前最佳实践脚本的TF文件（如main.tf），确保其中（也可以是其他同级目录下的TF文件）包含部署资源所需的provider版本声明和华为云鉴权信息。
配置介绍参考[部署华为云资源前的准备工作](../../introductions/prepare_before_deploy.md)一文中的介绍。

### 2. 创建AAD实例（可选）

当您没有现成的AAD实例时，需要先创建AAD实例。在TF文件（如main.tf）中添加以下脚本：

```hcl
# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下创建AAD实例
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
- **resource_region**：资源区域，可选值包括north_china（华北）、east_china（华东）、asia_pacific（亚太）。
- **instance_access_type**：接入类型，0表示网站，1表示IP接入。
- **duration**：订购时长。
- **amount**：购买实例数量。
- **instance_name**：实例名称。
- **period_type**：订购周期类型，2表示月，3表示年。
- **service_bandwidth**：业务带宽（Mbps）。
- **basic_bandwidth**：基础带宽（Mbps），当resource_region为north_china或east_china时必填，为asia_pacific时留空。
- **elastic_bandwidth**：弹性带宽（Mbps），当resource_region为north_china或east_china时必填，为asia_pacific时留空。
- **basic_qps**：业务QPS，当instance_access_type为0时必填，为1时留空。
- **forwarding_rule**：转发规则数，当instance_access_type为1时必填，为0时留空。
- **protected_domain**：防护域名数，当instance_access_type为0时必填，为1时留空。
- **elastic_service_bandwidth_type**：弹性业务带宽类型，2表示日95，3表示月95。
- **elastic_service_bandwidth**：弹性业务带宽增量。
- **protection_package**：防护包，可选值包括basic（保险防护）、unlimited（无限防护）。
- **enterprise_project_id**：企业项目ID。

### 3. 配置黑白名单

创建AAD实例后（或使用已有实例），需要为其添加IP黑白名单。在TF文件（如main.tf）中添加以下脚本：

```hcl
# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下添加IP黑名单
resource "huaweicloud_aad_black_white_list" "black" {
  instance_id = var.instance_id != "" ? var.instance_id : try(huaweicloud_aad_instance.test[0].id, "")
  type        = "black"
  ips         = var.blacklist_ips
}

# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下添加IP白名单
resource "huaweicloud_aad_black_white_list" "white" {
  instance_id = var.instance_id != "" ? var.instance_id : try(huaweicloud_aad_instance.test[0].id, "")
  type        = "white"
  ips         = var.whitelist_ips
}
```

**参数说明**：
- **instance_id**：AAD实例ID，当您已有实例时，直接指定；否则通过`huaweicloud_aad_instance.test[0].id`获取。
- **type**：名单类型，black表示黑名单，white表示白名单。
- **ips**：需要添加的IP地址列表。

### 4. 预设资源部署所需的入参

本实践中，部分资源、数据源使用了输入变量对配置内容进行赋值，这些输入参数在后续部署时需要手工输入。
同时，Terraform提供了通过`tfvars`文件预设这些配置的方法，可以避免每次执行时重复输入。

在工作目录下创建`terraform.tfvars`文件，示例内容如下：

```hcl
# 根据脚本变量填写；敏感信息使用占位符
region_name = "cn-north-4"
access_key  = "your-access-key"
secret_key  = "your-secret-key"

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

1. 命令行参数：`terraform apply -var="vpc_name=my-vpc"`
2. 环境变量：`export TF_VAR_vpc_name=my-vpc`
3. 自定义命名的变量文件：`terraform apply -var-file="custom.tfvars"`

> 注意：如果同一个变量通过多种方式进行设置，Terraform会按照以下优先级使用变量值：命令行参数 > 变量文件 > 环境变量 > 默认值。

### 5. 初始化并应用Terraform配置

完成以上脚本配置后，执行以下步骤来创建资源：

1. 运行 `terraform init` 初始化环境
2. 运行 `terraform plan` 查看资源创建计划
3. 确认资源计划无误后，运行 `terraform apply` 开始创建AAD实例及黑白名单
4. 运行 `terraform show` 查看已创建的AAD实例及黑白名单

## 参考信息

- [华为云AAD产品文档](https://support.huaweicloud.com/aad/index.html)
- [华为云Provider文档](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs)
- [AAD黑白名单最佳实践源码参考](https://github.com/huaweicloud/terraform-provider-huaweicloud/tree/master/examples/aad/black-white-lists)
