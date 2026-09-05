# 部署DDM账号

## 应用场景

分布式数据库中间件（DDM）是华为云提供的分布式关系型数据库中间件，兼容MySQL协议，支持分库分表、读写分离等能力。在实际业务中，您可能需要为DDM实例创建独立的数据库账号，用于访问和管理逻辑库。

本最佳实践将介绍如何使用Terraform自动化部署DDM实例及账号，包括创建VPC、子网、安全组，配置引擎、规格与节点数量，并创建具备指定权限的DDM账号。

## 相关资源/数据源

本最佳实践涉及以下主要资源和数据源：

### 数据源

- [可用分区（data.huaweicloud_availability_zones）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/data-sources/availability_zones)
- [DDM引擎（data.huaweicloud_ddm_engines）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/data-sources/ddm_engines)
- [DDM规格（data.huaweicloud_ddm_flavors）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/data-sources/ddm_flavors)

### 资源

- [虚拟私有云VPC（huaweicloud_vpc）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/vpc)
- [子网（huaweicloud_vpc_subnet）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/vpc_subnet)
- [安全组（huaweicloud_networking_secgroup）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/networking_secgroup)
- [DDM实例（huaweicloud_ddm_instance）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/ddm_instance)
- [随机密码（random_password）](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/password)
- [DDM账号（huaweicloud_ddm_account）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/ddm_account)

### 资源/数据源依赖关系

```
data.huaweicloud_availability_zones
    └── huaweicloud_ddm_instance

data.huaweicloud_ddm_engines
    └── data.huaweicloud_ddm_flavors
        └── huaweicloud_ddm_instance

huaweicloud_vpc
    └── huaweicloud_vpc_subnet
        └── huaweicloud_ddm_instance

huaweicloud_networking_secgroup
    └── huaweicloud_ddm_instance

huaweicloud_ddm_instance
    └── huaweicloud_ddm_account

random_password
    └── huaweicloud_ddm_account
```

## 操作步骤

### 1. 脚本准备

在指定工作空间中准备好用于编写当前最佳实践脚本的TF文件（如main.tf），确保其中（也可以是其他同级目录下的TF文件）包含部署资源所需的provider版本声明和华为云鉴权信息。
配置介绍参考[部署华为云资源前的准备工作](../../introductions/prepare_before_deploy.md)一文中的介绍。

### 2. 查询可用分区

在TF文件（如main.tf）中添加以下脚本，用于在未指定可用分区时查询可用的可用分区列表：

```hcl
# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下查询可用分区
data "huaweicloud_availability_zones" "test" {
  count = length(var.availability_zones) == 0 ? 1 : 0
}
```

**参数说明**：
- **count**：当输入变量availability_zones为空列表时，创建该数据源以查询可用分区。

### 3. 创建VPC

在TF文件（如main.tf）中添加以下脚本，用于创建VPC：

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
- **name**：通过引用输入变量vpc_name进行赋值。
- **cidr**：通过引用输入变量vpc_cidr进行赋值，默认值为"192.168.0.0/16"。

### 4. 创建子网

在TF文件（如main.tf）中添加以下脚本，用于创建子网：

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
  description = "The gateway IP of the subnet"
  type        = string
  default     = ""
  nullable    = false
}

resource "huaweicloud_vpc_subnet" "test" {
  vpc_id     = huaweicloud_vpc.test.id
  name       = var.subnet_name
  cidr       = var.subnet_cidr == "" ? cidrsubnet(huaweicloud_vpc.test.cidr, 8, 0) : var.subnet_cidr
  gateway_ip = var.subnet_gateway_ip == "" ? cidrhost(cidrsubnet(huaweicloud_vpc.test.cidr, 8, 0), 1) : var.subnet_gateway_ip
}
```

**参数说明**：
- **vpc_id**：通过引用huaweicloud_vpc.test的ID进行赋值。
- **name**：通过引用输入变量subnet_name进行赋值。
- **cidr**：通过引用输入变量subnet_cidr进行赋值，若未指定则自动从VPC的CIDR中划分。
- **gateway_ip**：通过引用输入变量subnet_gateway_ip进行赋值，若未指定则自动计算网关地址。

### 5. 创建安全组

在TF文件（如main.tf）中添加以下脚本，用于创建安全组：

```hcl
# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下创建安全组
variable "security_group_name" {
  description = "The name of the security group"
  type        = string
}

resource "huaweicloud_networking_secgroup" "test" {
  name                 = var.security_group_name
  delete_default_rules = true
}
```

**参数说明**：
- **name**：通过引用输入变量security_group_name进行赋值。
- **delete_default_rules**：设置为true，删除默认安全组规则。

### 6. 查询DDM引擎和规格

在TF文件（如main.tf）中添加以下脚本，用于在未指定引擎和规格时查询可用的DDM引擎和规格：

```hcl
# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下查询DDM引擎
data "huaweicloud_ddm_engines" "test" {
  count = var.instance_engine_id == "" ? 1 : 0
}

# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下查询DDM规格
data "huaweicloud_ddm_flavors" "test" {
  count = var.instance_flavor_id == "" ? 1 : 0

  engine_id = var.instance_engine_id == "" ? try(data.huaweicloud_ddm_engines.test[0].engines[0].id, null) : var.instance_engine_id
}
```

**参数说明**：
- **count**：当输入变量instance_engine_id或instance_flavor_id为空时，创建数据源以查询默认的引擎和规格。
- **engine_id**：通过引用输入变量instance_engine_id或data.huaweicloud_ddm_engines.test的查询结果进行赋值。

### 7. 创建DDM实例

在TF文件（如main.tf）中添加以下脚本，用于创建DDM实例：

```hcl
# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下创建DDM实例
variable "availability_zones" {
  description = "The availability zones to which the DDM instance belongs"
  type        = list(string)
  default     = []
  nullable    = false
}

variable "instance_engine_id" {
  description = "The engine ID of the DDM instance"
  type        = string
  default     = ""
  nullable    = false
}

variable "instance_flavor_id" {
  description = "The flavor ID of the DDM instance"
  type        = string
  default     = ""
  nullable    = false
}

variable "instance_name" {
  description = "The name of the DDM instance"
  type        = string
}

variable "instance_node_num" {
  description = "The number of nodes in the DDM instance"
  type        = number
  default     = 2
}

variable "instance_parameters" {
  description = "The parameters of the DDM instance"

  type = list(object({
    name  = string
    value = string
  }))

  default = []
}

resource "huaweicloud_ddm_instance" "test" {
  name               = var.instance_name
  availability_zones = length(var.availability_zones) == 0 ? try(slice(data.huaweicloud_availability_zones.test[0].names, 0, 1)) : var.availability_zones
  engine_id          = var.instance_engine_id == "" ? try(data.huaweicloud_ddm_engines.test[0].engines[0].id, null) : var.instance_engine_id
  flavor_id          = var.instance_flavor_id == "" ? try(data.huaweicloud_ddm_flavors.test[0].flavors[0].id, null) : var.instance_flavor_id
  vpc_id             = huaweicloud_vpc.test.id
  subnet_id          = huaweicloud_vpc_subnet.test.id
  security_group_id  = huaweicloud_networking_secgroup.test.id
  node_num           = var.instance_node_num

  dynamic "parameters" {
    for_each = var.instance_parameters

    content {
      name  = parameters.value.name
      value = parameters.value.value
    }
  }
}
```

**参数说明**：
- **name**：通过引用输入变量instance_name进行赋值。
- **availability_zones**：通过引用输入变量availability_zones或data.huaweicloud_availability_zones.test的查询结果进行赋值。
- **engine_id**：通过引用输入变量instance_engine_id或data.huaweicloud_ddm_engines.test的查询结果进行赋值。
- **flavor_id**：通过引用输入变量instance_flavor_id或data.huaweicloud_ddm_flavors.test的查询结果进行赋值。
- **vpc_id**：通过引用huaweicloud_vpc.test的ID进行赋值。
- **subnet_id**：通过引用huaweicloud_vpc_subnet.test的ID进行赋值。
- **security_group_id**：通过引用huaweicloud_networking_secgroup.test的ID进行赋值。
- **node_num**：通过引用输入变量instance_node_num进行赋值，默认值为2。
- **parameters**：通过引用输入变量instance_parameters动态设置实例参数。

### 8. 生成随机密码

在TF文件（如main.tf）中添加以下脚本，用于在未指定账号密码时生成随机密码：

```hcl
# 生成随机密码
variable "account_password" {
  description = "The password of the DDM account"
  sensitive   = true
  type        = string
  default     = ""
  nullable    = false
}

resource "random_password" "test" {
  count = var.account_password == "" ? 1 : 0

  length           = 12
  special          = true
  override_special = "!@#%^*-_+?"
  min_upper        = 1
  min_lower        = 1
  min_numeric      = 1
  min_special      = 1
}
```

**参数说明**：
- **count**：当输入变量account_password为空时，创建该资源以生成随机密码。
- **length**：密码长度为12。
- **special**：包含特殊字符。
- **override_special**：指定可用的特殊字符集合。
- **min_upper**：至少包含1个大写字母。
- **min_lower**：至少包含1个小写字母。
- **min_numeric**：至少包含1个数字。
- **min_special**：至少包含1个特殊字符。

### 9. 创建DDM账号

在TF文件（如main.tf）中添加以下脚本，用于创建DDM账号：

```hcl
# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下创建DDM账号
variable "account_name" {
  description = "The name of the DDM account"
  type        = string
}

variable "account_permissions" {
  description = "The basic permissions of the DDM account"
  type        = list(string)
  default     = ["SELECT"]
  nullable    = false
}

variable "account_description" {
  description = "The description of the DDM account"
  type        = string
  default     = ""
}

resource "huaweicloud_ddm_account" "test" {
  instance_id = huaweicloud_ddm_instance.test.id
  name        = var.account_name
  password    = var.account_password == "" ? try(random_password.test[0].result, null) : var.account_password
  permissions = var.account_permissions
  description = var.account_description
}
```

**参数说明**：
- **instance_id**：通过引用huaweicloud_ddm_instance.test的ID进行赋值。
- **name**：通过引用输入变量account_name进行赋值。
- **password**：通过引用输入变量account_password或random_password.test的生成结果进行赋值。
- **permissions**：通过引用输入变量account_permissions进行赋值，默认值为["SELECT"]。
- **description**：通过引用输入变量account_description进行赋值。

### 10. 预设资源部署所需的入参（可选）

本实践中，部分资源、数据源使用了输入变量对配置内容进行赋值，这些输入参数在后续部署时需要手工输入。
同时，Terraform提供了通过`tfvars`文件预设这些配置的方法，可以避免每次执行时重复输入。

在工作目录下创建`terraform.tfvars`文件，示例内容如下：

```hcl
# 根据脚本变量填写；敏感信息使用占位符
vpc_name            = "your_vpc_name"
subnet_name         = "your_subnet_name"
security_group_name = "your_security_group_name"
instance_name       = "your_instance_name"
account_name        = "your_account_name"
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
3. 确认资源计划无误后，运行 `terraform apply` 开始创建DDM实例及账号
4. 运行 `terraform show` 查看已创建的DDM实例及账号

## 参考信息

- [华为云DDM产品文档](https://support.huaweicloud.com/ddm/index.html)
- [华为云Provider文档](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs)
- [DDM账号最佳实践源码参考](https://github.com/huaweicloud/terraform-provider-huaweicloud/tree/master/examples/ddm/ddm-account)
