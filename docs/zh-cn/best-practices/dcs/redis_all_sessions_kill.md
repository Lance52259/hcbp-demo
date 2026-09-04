# 部署Redis实例全会话清理

## 应用场景

分布式缓存服务（Distributed Cache Service, DCS）是华为云提供的高性能、高可用的内存数据库服务，支持Redis、Memcached等主流缓存引擎。当Redis实例中存在大量异常或空闲的客户端会话时，可能会占用连接资源，影响实例的稳定性和性能。

本最佳实践将介绍如何使用Terraform在华为云上部署一个DCS Redis单机实例，并通过`huaweicloud_dcs_all_sessions_kill`资源清理该实例的所有客户端会话。该实践涵盖了VPC、子网、可用区、实例规格等资源的自动创建与配置，帮助您快速掌握DCS实例会话管理的自动化部署流程。

## 相关资源/数据源

本最佳实践涉及以下主要资源和数据源：

### 数据源

- [可用区（data.huaweicloud_availability_zones）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/data-sources/availability_zones)
- [DCS实例规格（data.huaweicloud_dcs_flavors）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/data-sources/dcs_flavors)
- [DCS实例分片信息（data.huaweicloud_dcs_instance_shards）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/data-sources/dcs_instance_shards)

### 资源

- [虚拟私有云VPC（huaweicloud_vpc）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/vpc)
- [子网（huaweicloud_vpc_subnet）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/vpc_subnet)
- [随机密码（random_password）](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/password)
- [DCS Redis实例（huaweicloud_dcs_instance）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/dcs_instance)
- [DCS全会话清理（huaweicloud_dcs_all_sessions_kill）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/dcs_all_sessions_kill)

### 资源/数据源依赖关系

```
huaweicloud_vpc
    └── huaweicloud_vpc_subnet
          └── huaweicloud_dcs_instance
                ├── data.huaweicloud_dcs_instance_shards
                └── huaweicloud_dcs_all_sessions_kill
```

## 操作步骤

### 1. 脚本准备

在指定工作空间中准备好用于编写当前最佳实践脚本的TF文件（如main.tf），确保其中（也可以是其他同级目录下的TF文件）包含部署资源所需的provider版本声明和华为云鉴权信息。
配置介绍参考[部署华为云资源前的准备工作](../../introductions/prepare_before_deploy.md)一文中的介绍。

### 2. 创建VPC

在TF文件（如main.tf）中添加以下脚本：

```hcl
# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下创建VPC
variable "vpc_name" {
  description = "The name of the VPC"
  type        = string
}

variable "vpc_cidr" {
  description = "The CIDR block of the VPC"
  type        = string
  default     = "192.168.0.0/16"
}

resource "huaweicloud_vpc" "test" {
  name = var.vpc_name
  cidr = var.vpc_cidr
}
```

**参数说明**：
- **name**：通过引用输入变量 vpc_name 进行赋值，用于指定VPC的名称。
- **cidr**：通过引用输入变量 vpc_cidr 进行赋值，用于指定VPC的CIDR网段。

### 3. 创建子网

在TF文件（如main.tf）中添加以下脚本：

```hcl
# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下创建子网
variable "subnet_name" {
  description = "The name of the subnet"
  type        = string
}

variable "subnet_cidr" {
  description = "The CIDR block of the subnet"
  type        = string
  default     = ""
  nullable    = false
}

variable "subnet_gateway_ip" {
  description = "The gateway IP address of the subnet"
  type        = string
  default     = ""
  nullable    = false
}

resource "huaweicloud_vpc_subnet" "test" {
  vpc_id     = huaweicloud_vpc.test.id
  name       = var.subnet_name
  cidr       = var.subnet_cidr != "" ? var.subnet_cidr : cidrsubnet(huaweicloud_vpc.test.cidr, 8, 0)
  gateway_ip = (var.subnet_gateway_ip != "" ? var.subnet_gateway_ip :
  cidrhost(cidrsubnet(huaweicloud_vpc.test.cidr, 8, 0), 1))
}
```

**参数说明**：
- **vpc_id**：通过引用 huaweicloud_vpc.test.id 进行赋值，指定子网所属的VPC。
- **name**：通过引用输入变量 subnet_name 进行赋值，用于指定子网的名称。
- **cidr**：通过引用输入变量 subnet_cidr 进行赋值，若未设置则自动从VPC的CIDR中划分。
- **gateway_ip**：通过引用输入变量 subnet_gateway_ip 进行赋值，若未设置则自动计算网关地址。

### 4. 查询可用区

在TF文件（如main.tf）中添加以下脚本：

```hcl
# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下查询可用区
variable "availability_zone" {
  description = "The availability zone to which the Redis single instance belongs"
  type        = string
  default     = ""
  nullable    = false
}

data "huaweicloud_availability_zones" "test" {
  count = var.availability_zone == "" ? 1 : 0
}
```

**参数说明**：
- **count**：当未指定可用区时，通过条件表达式创建数据源，用于查询可用的可用区列表。

### 5. 查询DCS实例规格

在TF文件（如main.tf）中添加以下脚本：

```hcl
# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下查询DCS实例规格
data "huaweicloud_dcs_flavors" "test" {
  count = var.instance_flavor_id == "" ? 1 : 0

  engine         = "Redis"
  capacity       = var.instance_capacity
  engine_version = var.instance_engine_version
}
```

**参数说明**：
- **count**：当未指定实例规格ID时，通过条件表达式创建数据源，用于查询可用的DCS实例规格。
- **engine**：指定缓存引擎为Redis。
- **capacity**：通过引用输入变量 instance_capacity 进行赋值，指定实例容量。
- **engine_version**：通过引用输入变量 instance_engine_version 进行赋值，指定引擎版本。

### 6. 生成随机密码

在TF文件（如main.tf）中添加以下脚本：

```hcl
# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下生成随机密码
variable "instance_password" {
  description = "The password for the Redis instance"
  type        = string
  sensitive   = true
  default     = null
}

resource "random_password" "test" {
  count = var.instance_password == "" ? 1 : 0

  length           = 12
  special          = true
  override_special = "!@%^*-_=+"
  min_upper        = 1
  min_lower        = 1
  min_numeric      = 1
  min_special      = 1
}
```

**参数说明**：
- **count**：当未指定实例密码时，通过条件表达式创建随机密码资源。
- **length**：指定生成的密码长度为12。
- **special**：指定生成的密码包含特殊字符。
- **override_special**：指定允许的特殊字符集合。
- **min_upper**：指定至少包含1个大写字母。
- **min_lower**：指定至少包含1个小写字母。
- **min_numeric**：指定至少包含1个数字。
- **min_special**：指定至少包含1个特殊字符。

### 7. 创建DCS Redis实例

在TF文件（如main.tf）中添加以下脚本：

```hcl
# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下创建DCS Redis实例
variable "instance_name" {
  description = "The name of the Redis single instance"
  type        = string
}

variable "instance_flavor_id" {
  description = "The flavor ID of the Redis single instance"
  type        = string
  default     = ""
}

variable "instance_capacity" {
  description = "The capacity of the Redis instance (in GB)"
  type        = number
  default     = 4
}

variable "instance_engine_version" {
  description = "The engine version of the Redis single instance"
  type        = string
  default     = "5.0"
}

resource "huaweicloud_dcs_instance" "test" {
  name               = var.instance_name
  engine             = "Redis"
  engine_version     = var.instance_engine_version
  capacity           = var.instance_capacity
  vpc_id             = huaweicloud_vpc.test.id
  subnet_id          = huaweicloud_vpc_subnet.test.id
  password           = (var.instance_password != "" ? var.instance_password :
  try(random_password.test[0].result, null))
  flavor             = (var.instance_flavor_id != "" ? var.instance_flavor_id :
  try(data.huaweicloud_dcs_flavors.test[0].flavors[0].name, null))
  availability_zones = (var.availability_zone != "" ? [var.availability_zone] :
  try(slice(data.huaweicloud_availability_zones.test[0].names, 0, 1), null))
}
```

**参数说明**：
- **name**：通过引用输入变量 instance_name 进行赋值，用于指定DCS实例的名称。
- **engine**：指定缓存引擎为Redis。
- **engine_version**：通过引用输入变量 instance_engine_version 进行赋值，指定引擎版本。
- **capacity**：通过引用输入变量 instance_capacity 进行赋值，指定实例容量。
- **vpc_id**：通过引用 huaweicloud_vpc.test.id 进行赋值，指定实例所属的VPC。
- **subnet_id**：通过引用 huaweicloud_vpc_subnet.test.id 进行赋值，指定实例所属的子网。
- **password**：通过引用输入变量 instance_password 进行赋值，若未设置则使用随机生成的密码。
- **flavor**：通过引用输入变量 instance_flavor_id 进行赋值，若未设置则使用查询到的第一个规格。
- **availability_zones**：通过引用输入变量 availability_zone 进行赋值，若未设置则使用查询到的第一个可用区。

### 8. 查询DCS实例分片信息

在TF文件（如main.tf）中添加以下脚本：

```hcl
# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下查询DCS实例分片信息
data "huaweicloud_dcs_instance_shards" "test" {
  instance_id = huaweicloud_dcs_instance.test.id
}
```

**参数说明**：
- **instance_id**：通过引用 huaweicloud_dcs_instance.test.id 进行赋值，指定要查询分片信息的DCS实例。

### 9. 清理DCS实例所有客户端会话

在TF文件（如main.tf）中添加以下脚本：

```hcl
# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下清理DCS实例所有客户端会话
variable "kill_all_nodes" {
  description = "Whether to kill all client sessions of all nodes. Valid values: true, false"
  type        = string
  default     = "false"
}

locals {
  replication_list = try(data.huaweicloud_dcs_instance_shards.test.group_list[0].replication_list, [])
}

resource "huaweicloud_dcs_all_sessions_kill" "test" {
  instance_id    = huaweicloud_dcs_instance.test.id
  kill_all_nodes = var.kill_all_nodes
  node_id        = try(local.replication_list[0].node_id, "")
}
```

**参数说明**：
- **instance_id**：通过引用 huaweicloud_dcs_instance.test.id 进行赋值，指定要清理会话的DCS实例。
- **kill_all_nodes**：通过引用输入变量 kill_all_nodes 进行赋值，指定是否清理所有节点的会话。
- **node_id**：通过引用 local.replication_list 中的节点ID进行赋值，当kill_all_nodes为false时用于指定节点。

### 10. 预设资源部署所需的入参（可选）

本实践中，部分资源、数据源使用了输入变量对配置内容进行赋值，这些输入参数在后续部署时需要手工输入。
同时，Terraform提供了通过`tfvars`文件预设这些配置的方法，可以避免每次执行时重复输入。

在工作目录下创建`terraform.tfvars`文件，示例内容如下：

```hcl
# 根据脚本变量填写；敏感信息使用占位符
vpc_name      = "tf_test_dcs_instance_vpc"
subnet_name   = "tf_test_dcs_instance_subnet"
instance_name = "tf_test_dcs_instance"
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

### 11. 初始化并应用Terraform配置

完成以上脚本配置后，执行以下步骤来创建资源：

1. 运行 `terraform init` 初始化环境
2. 运行 `terraform plan` 查看资源创建计划
3. 确认资源计划无误后，运行 `terraform apply` 开始创建DCS Redis实例并清理其客户端会话
4. 运行 `terraform show` 查看已创建的DCS Redis实例及会话清理结果

## 参考信息

- [华为云DCS产品文档](https://support.huaweicloud.com/dcs/index.html)
- [华为云Provider文档](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs)
- [DCS Redis实例全会话清理最佳实践源码参考](https://github.com/huaweicloud/terraform-provider-huaweicloud/tree/master/examples/dcs/redis-all-sessions-kill)
