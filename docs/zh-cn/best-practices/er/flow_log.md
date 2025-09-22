# 部署流日志

## 应用场景

企业路由器（Enterprise Router, ER）是华为云提供的高性能、高可用的企业级路由器服务，支持多VPC互通、专线接入、VPN连接等企业级网络功能。ER服务提供灵活的路由策略和丰富的网络连接能力，满足企业复杂的网络架构需求。

ER流日志是ER服务的重要功能，用于记录和监控企业路由器上的网络流量信息，包括数据包的源地址、目标地址、协议类型、端口等详细信息。通过流日志，企业可以分析网络流量模式、监控网络性能、进行安全审计和故障排查。本最佳实践将介绍如何使用Terraform自动化部署流日志，包括VPC创建、ER实例创建、VPC连接、LTS日志组创建和流日志配置。

## 相关资源/数据源

本最佳实践涉及以下主要资源和数据源：

### 数据源

- [ER可用区列表查询数据源（data.huaweicloud_er_availability_zones）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/data-sources/er_availability_zones)

### 资源

- [VPC资源（huaweicloud_vpc）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/vpc)
- [VPC子网资源（huaweicloud_vpc_subnet）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/vpc_subnet)
- [ER实例资源（huaweicloud_er_instance）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/er_instance)
- [ER VPC连接资源（huaweicloud_er_vpc_attachment）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/er_vpc_attachment)
- [LTS日志组资源（huaweicloud_lts_group）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/lts_group)
- [LTS日志流资源（huaweicloud_lts_stream）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/lts_stream)
- [ER流日志资源（huaweicloud_er_flow_log）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/er_flow_log)

### 资源/数据源依赖关系

```
data.huaweicloud_er_availability_zones.test
    └── huaweicloud_er_instance.test

huaweicloud_vpc.test
    ├── huaweicloud_vpc_subnet.test
    │   └── huaweicloud_er_vpc_attachment.test
    └── huaweicloud_er_vpc_attachment.test

huaweicloud_er_instance.test
    └── huaweicloud_er_vpc_attachment.test
        └── huaweicloud_er_flow_log.test

huaweicloud_lts_group.test
    └── huaweicloud_lts_stream.test
        └── huaweicloud_er_flow_log.test
```

## 操作步骤

### 1. 脚本准备

在指定工作空间中准备好用于编写当前最佳实践脚本的TF文件（如main.tf），确保其中（也可以是其他同级目录下的TF文件）包含部署资源所需的provider版本声明和华为云鉴权信息。
配置介绍参考[部署华为云资源前的准备工作](../introductions/prepare_before_deploy.md)一文中的介绍。

### 2. 通过数据源查询ER可用区信息

在TF文件（如main.tf）中添加以下脚本以告知Terraform进行一次数据源查询，其查询结果用于创建ER实例：

```hcl
# 获取指定region（region参数缺省时默认继承当前provider块中所指定的region）下所有的ER可用区信息，用于创建ER实例
data "huaweicloud_er_availability_zones" "test" {}
```

**参数说明**：
- 无需额外参数，数据源会自动获取当前region下的所有ER可用区信息

### 3. 创建VPC

在TF文件（如main.tf）中添加以下脚本以告知Terraform创建VPC资源：

```hcl
variable "vpc_name" {
  description = "VPC名称"
  type        = string
}

variable "vpc_cidr" {
  description = "VPC的CIDR块"
  type        = string
  default     = "192.168.0.0/16"
}

# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下创建VPC资源
resource "huaweicloud_vpc" "test" {
  name = var.vpc_name
  cidr = var.vpc_cidr
}
```

**参数说明**：
- **name**：VPC名称，通过引用输入变量vpc_name进行赋值
- **cidr**：VPC的CIDR块，通过引用输入变量vpc_cidr进行赋值

### 4. 创建VPC子网

在TF文件（如main.tf）中添加以下脚本以告知Terraform创建VPC子网资源：

```hcl
variable "subnet_name" {
  description = "VPC子网名称"
  type        = string
}

variable "subnet_cidr" {
  description = "VPC子网的CIDR块"
  type        = string
  default     = ""
}

variable "subnet_gateway_ip" {
  description = "VPC子网的网关IP"
  type        = string
  default     = ""
}

# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下创建VPC子网资源
resource "huaweicloud_vpc_subnet" "test" {
  vpc_id     = huaweicloud_vpc.test.id
  name       = var.subnet_name
  cidr       = var.subnet_cidr == "" ? cidrsubnet(huaweicloud_vpc.test.cidr, 8, 0) : var.subnet_cidr
  gateway_ip = var.subnet_gateway_ip == "" ? cidrhost(cidrsubnet(huaweicloud_vpc.test.cidr, 8, 0), 1) : var.subnet_gateway_ip
}
```

**参数说明**：
- **vpc_id**：VPC ID，通过引用VPC资源（huaweicloud_vpc.test）的ID进行赋值
- **name**：子网名称，通过引用输入变量subnet_name进行赋值
- **cidr**：子网CIDR块，优先使用输入变量，如果为空则通过cidrsubnet函数计算
- **gateway_ip**：网关IP，优先使用输入变量，如果为空则通过cidrhost函数计算

### 5. 创建ER实例

在TF文件（如main.tf）中添加以下脚本以告知Terraform创建ER实例资源：

```hcl
variable "er_instance_name" {
  description = "ER实例名称"
  type        = string
}

variable "er_instance_asn" {
  description = "ER实例ASN号"
  type        = number
}

# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下创建ER实例资源
resource "huaweicloud_er_instance" "test" {
  availability_zones = slice(data.huaweicloud_er_availability_zones.test.names, 0, 1)
  name               = var.er_instance_name
  asn                = var.er_instance_asn
}
```

**参数说明**：
- **availability_zones**：可用区列表，使用ER可用区列表查询数据源的第一个结果
- **name**：实例名称，通过引用输入变量er_instance_name进行赋值
- **asn**：ASN号，通过引用输入变量er_instance_asn进行赋值

### 6. 创建ER VPC连接

在TF文件（如main.tf）中添加以下脚本以告知Terraform创建ER VPC连接资源：

```hcl
variable "er_vpc_attachment_name" {
  description = "ER VPC连接名称"
  type        = string
}

variable "er_vpc_attachment_auto_create_vpc_routes" {
  description = "是否自动创建VPC路由"
  type        = bool
  default     = true
}

# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下创建ER VPC连接资源
resource "huaweicloud_er_vpc_attachment" "test" {
  instance_id = huaweicloud_er_instance.test.id
  vpc_id      = huaweicloud_vpc.test.id
  subnet_id   = huaweicloud_vpc_subnet.test.id

  name                   = var.er_vpc_attachment_name
  auto_create_vpc_routes = var.er_vpc_attachment_auto_create_vpc_routes
}
```

**参数说明**：
- **instance_id**：ER实例ID，通过引用ER实例资源（huaweicloud_er_instance.test）的ID进行赋值
- **vpc_id**：VPC ID，通过引用VPC资源（huaweicloud_vpc.test）的ID进行赋值
- **subnet_id**：子网ID，通过引用VPC子网资源（huaweicloud_vpc_subnet.test）的ID进行赋值
- **name**：连接名称，通过引用输入变量er_vpc_attachment_name进行赋值
- **auto_create_vpc_routes**：自动创建VPC路由，通过引用输入变量er_vpc_attachment_auto_create_vpc_routes进行赋值

### 7. 创建LTS日志组

在TF文件（如main.tf）中添加以下脚本以告知Terraform创建LTS日志组资源：

```hcl
variable "lts_group_name" {
  description = "LTS日志组名称"
  type        = string
}

variable "lts_group_ttl_in_days" {
  description = "日志保存天数"
  type        = number
  default     = 7
}

# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下创建LTS日志组资源
resource "huaweicloud_lts_group" "test" {
  group_name  = var.lts_group_name
  ttl_in_days = var.lts_group_ttl_in_days
}
```

**参数说明**：
- **group_name**：日志组名称，通过引用输入变量lts_group_name进行赋值
- **ttl_in_days**：日志保存天数，通过引用输入变量lts_group_ttl_in_days进行赋值

### 8. 创建LTS日志流

在TF文件（如main.tf）中添加以下脚本以告知Terraform创建LTS日志流资源：

```hcl
variable "lts_stream_name" {
  description = "LTS日志流名称"
  type        = string
}

# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下创建LTS日志流资源
resource "huaweicloud_lts_stream" "test" {
  group_id    = huaweicloud_lts_group.test.id
  stream_name = var.lts_stream_name
}
```

**参数说明**：
- **group_id**：日志组ID，通过引用LTS日志组资源（huaweicloud_lts_group.test）的ID进行赋值
- **stream_name**：日志流名称，通过引用输入变量lts_stream_name进行赋值

### 9. 创建ER流日志

在TF文件（如main.tf）中添加以下脚本以告知Terraform创建ER流日志资源：

```hcl
variable "er_flow_log_name" {
  description = "ER流日志名称"
  type        = string
}

variable "er_flow_log_store_type" {
  description = "流日志存储类型"
  type        = string
  default     = "LTS"
}

variable "er_flow_log_resource_type" {
  description = "流日志收集的资源类型"
  type        = string
  default     = "attachment"
}

# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下创建ER流日志资源
resource "huaweicloud_er_flow_log" "test" {
  name           = var.er_flow_log_name
  instance_id    = huaweicloud_er_instance.test.id
  log_store_type = var.er_flow_log_store_type
  log_group_id   = huaweicloud_lts_group.test.id
  log_stream_id  = huaweicloud_lts_stream.test.id
  resource_type  = var.er_flow_log_resource_type
  resource_id    = huaweicloud_er_vpc_attachment.test.id
}
```

**参数说明**：
- **name**：流日志名称，通过引用输入变量er_flow_log_name进行赋值
- **instance_id**：ER实例ID，通过引用ER实例资源（huaweicloud_er_instance.test）的ID进行赋值
- **log_store_type**：日志存储类型，通过引用输入变量er_flow_log_store_type进行赋值
- **log_group_id**：日志组ID，通过引用LTS日志组资源（huaweicloud_lts_group.test）的ID进行赋值
- **log_stream_id**：日志流ID，通过引用LTS日志流资源（huaweicloud_lts_stream.test）的ID进行赋值
- **resource_type**：资源类型，通过引用输入变量er_flow_log_resource_type进行赋值
- **resource_id**：资源ID，通过引用ER VPC连接资源（huaweicloud_er_vpc_attachment.test）的ID进行赋值

### 10. 预设资源部署所需的入参（可选）

本实践中，部分资源、数据源使用了输入变量对配置内容进行赋值，这些输入参数在后续部署时需要手工输入。
同时，Terraform提供了通过`tfvars`文件预设这些配置的方法，可以避免每次执行时重复输入。

在工作目录下创建`terraform.tfvars`文件，示例内容如下：

```hcl
# 认证信息
region_name = "cn-north-4"
access_key  = "your-access-key"
secret_key  = "your-secret-key"

# 网络配置
vpc_name    = "tf_test_er_instance_vpc"
vpc_cidr    = "192.168.0.0/16"
subnet_name = "tf_test_er_instance_subnet"

# ER实例配置
er_instance_name = "tf_test_er_instance"
er_instance_asn  = 64512

# ER VPC连接配置
er_vpc_attachment_name = "tf_test_er_vpc_attachment"

# LTS日志配置
lts_group_name  = "tf_test_lts_group"
lts_stream_name = "tf_test_lts_stream"

# ER流日志配置
er_flow_log_name = "tf_test_er_flow_log"
```

**使用方法**：

1. 将上述内容保存为工作目录下的`terraform.tfvars`文件（该文件名可使用户在执行terraform命令时自动导入该`tfvars`文件中的内容，其他命名则需要在tfvars前补充`.auto`定义，如`variables.auto.tfvars`）
2. 根据实际需要修改参数值
3. 执行`terraform plan`或`terraform apply`时，Terraform会自动读取该文件中的变量值

除了使用`terraform.tfvars`文件外，还可以通过以下方式设置变量值：

1. 命令行参数：`terraform apply -var="vpc_name=my-vpc" -var="er_instance_name=my-er"`
2. 环境变量：`export TF_VAR_vpc_name=my-vpc`
3. 自定义命名的变量文件：`terraform apply -var-file="custom.tfvars"`

> 注意：如果同一个变量通过多种方式进行设置，Terraform会按照以下优先级使用变量值：命令行参数 > 变量文件 > 环境变量 > 默认值。

### 11. 初始化并应用Terraform配置

完成以上脚本配置后，执行以下步骤来创建资源：

1. 运行 `terraform init` 初始化环境
2. 运行 `terraform plan` 查看资源创建计划
3. 确认资源计划无误后，运行 `terraform apply` 开始创建流日志
4. 运行 `terraform show` 查看已创建的流日志

## 参考信息

- [华为云ER产品文档](https://support.huaweicloud.com/er/index.html)
- [华为云LTS产品文档](https://support.huaweicloud.com/lts/index.html)
- [华为云Provider文档](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs)
- [ER最佳实践源码参考](https://github.com/huaweicloud/terraform-provider-huaweicloud/tree/master/examples/er)
