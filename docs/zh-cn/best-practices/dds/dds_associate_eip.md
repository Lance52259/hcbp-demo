# 部署DDS实例绑定弹性公网IP

## 应用场景

文档数据库服务（Document Database Service，简称DDS）是华为云提供的高性能、高可靠、高安全的云数据库服务，完全兼容MongoDB协议，适用于各种场景下的非结构化数据存储需求。在实际业务中，为了便于从公网访问DDS实例，通常需要为DDS实例绑定弹性公网IP（EIP），实现远程管理和数据交互。

本最佳实践将介绍如何使用Terraform创建DDS实例并为其绑定弹性公网IP，帮助您快速搭建一个可公网访问的文档数据库环境。

## 相关资源/数据源

本最佳实践涉及以下主要资源和数据源：

### 数据源

- [可用分区（huaweicloud_availability_zones）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/data-sources/availability_zones)
- [DDS实例信息（huaweicloud_dds_instances）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/data-sources/dds_instances)

### 资源

- [虚拟私有云（huaweicloud_vpc）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/vpc)
- [子网（huaweicloud_vpc_subnet）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/vpc_subnet)
- [安全组（huaweicloud_networking_secgroup）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/networking_secgroup)
- [弹性公网IP（huaweicloud_vpc_eip）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/vpc_eip)
- [DDS实例（huaweicloud_dds_instance）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/dds_instance)
- [DDS实例EIP绑定（huaweicloud_dds_instance_eip_associate）](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/dds_instance_eip_associate)

### 资源/数据源依赖关系

```
huaweicloud_vpc
    └── huaweicloud_vpc_subnet
huaweicloud_networking_secgroup
huaweicloud_vpc_eip
huaweicloud_dds_instance
    └── data.huaweicloud_dds_instances
        └── huaweicloud_dds_instance_eip_associate
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
- **name**：通过引用输入变量 vpc_name 进行赋值，指定VPC的名称。
- **cidr**：通过引用输入变量 vpc_cidr 进行赋值，指定VPC的CIDR网段。

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
- **vpc_id**：通过引用 huaweicloud_vpc.test.id 进行赋值，指定子网所属的VPC ID。
- **name**：通过引用输入变量 subnet_name 进行赋值，指定子网的名称。
- **cidr**：通过引用输入变量 subnet_cidr 进行赋值，当未设置时，自动从VPC的CIDR中划分一个子网。
- **gateway_ip**：通过引用输入变量 subnet_gateway_ip 进行赋值，当未设置时，自动计算子网的网关IP。

### 4. 创建安全组

在TF文件（如main.tf）中添加以下脚本：

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
- **name**：通过引用输入变量 security_group_name 进行赋值，指定安全组的名称。
- **delete_default_rules**：设置为 `true`，删除安全组默认规则，以便后续自定义规则。

### 5. 创建弹性公网IP

在TF文件（如main.tf）中添加以下脚本：

```hcl
# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下创建弹性公网IP
variable "eip_type" {
  description = "The type of the EIP"
  type        = string
  default     = "5_bgp"
}

variable "eip_bandwidth_name" {
  description = "The name of the EIP bandwidth"
  type        = string
}

variable "eip_bandwidth_size" {
  description = "The size of the EIP bandwidth in Mbit/s"
  type        = number
  default     = 5
}

variable "eip_bandwidth_charge_mode" {
  description = "The charge mode of the EIP bandwidth"
  type        = string
  default     = "traffic"
}

resource "huaweicloud_vpc_eip" "test" {
  publicip {
    type = var.eip_type
  }

  bandwidth {
    name        = var.eip_bandwidth_name
    share_type  = "PER"
    size        = var.eip_bandwidth_size
    charge_mode = var.eip_bandwidth_charge_mode
  }
}
```

**参数说明**：
- **publicip.type**：通过引用输入变量 eip_type 进行赋值，指定弹性公网IP的类型。
- **bandwidth.name**：通过引用输入变量 eip_bandwidth_name 进行赋值，指定带宽的名称。
- **bandwidth.share_type**：设置为 `PER`，表示独享带宽。
- **bandwidth.size**：通过引用输入变量 eip_bandwidth_size 进行赋值，指定带宽大小。
- **bandwidth.charge_mode**：通过引用输入变量 eip_bandwidth_charge_mode 进行赋值，指定带宽的计费模式。

### 6. 创建DDS实例

在TF文件（如main.tf）中添加以下脚本：

```hcl
# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下创建DDS实例
data "huaweicloud_availability_zones" "test" {
  count = var.availability_zone == "" ? 1 : 0
}

variable "availability_zone" {
  description = "The availability zone to which the DDS instance belongs"
  type        = string
  default     = ""
  nullable    = false
}

variable "instance_name" {
  description = "The name of the DDS instance"
  type        = string
}

variable "instance_mode" {
  description = "The type of the DDS instance"
  type        = string
  default     = "ReplicaSet"
}

variable "database_type" {
  description = "The database version type of the DDS instance"
  type        = string
  default     = "DDS-Community"
}

variable "database_version" {
  description = "The database version of the DDS instance"
  type        = string
  default     = "4.0"
}

variable "storage_engine" {
  description = "The storage engine of the DDS instance"
  type        = string
  default     = "wiredTiger"
}

variable "node_type" {
  description = "The type of the DDS instance node"
  type        = string
  default     = "replica"
}

variable "node_number" {
  description = "The number of nodes of the DDS instance"
  type        = number
  default     = 3
}

variable "node_spec_code" {
  description = "The spec code of the DDS instance node"
  type        = string
  default     = "dds.mongodb.s6.large.2.repset"
  nullable    = false
}

variable "node_storage_type" {
  description = "The storage type of the DDS instance node"
  type        = string
  default     = "ULTRAHIGH"
}

variable "node_size" {
  description = "The disk size of the node of the DDS instance"
  type        = number
  default     = 10
}

variable "node_list" {
  description = "The node IDs to be deleted of the DDS instance"
  type        = list(string)
  default     = null
}

resource "huaweicloud_dds_instance" "test" {
  name              = var.instance_name
  availability_zone = var.availability_zone == "" ? try(data.huaweicloud_availability_zones.test[0].names[0], null) : var.availability_zone
  vpc_id            = huaweicloud_vpc.test.id
  subnet_id         = huaweicloud_vpc_subnet.test.id
  security_group_id = huaweicloud_networking_secgroup.test.id
  mode              = var.instance_mode

  datastore {
    type           = var.database_type
    version        = var.database_version
    storage_engine = var.storage_engine
  }

  flavor {
    type      = var.node_type
    num       = var.node_number
    spec_code = var.node_spec_code
    storage   = var.node_storage_type
    size      = var.node_size
    node_list = var.node_list
  }
}
```

**参数说明**：
- **name**：通过引用输入变量 instance_name 进行赋值，指定DDS实例的名称。
- **availability_zone**：通过引用输入变量 availability_zone 进行赋值，当未设置时，通过数据源 huaweicloud_availability_zones 获取第一个可用分区。
- **vpc_id**：通过引用 huaweicloud_vpc.test.id 进行赋值，指定实例所属的VPC。
- **subnet_id**：通过引用 huaweicloud_vpc_subnet.test.id 进行赋值，指定实例所属的子网。
- **security_group_id**：通过引用 huaweicloud_networking_secgroup.test.id 进行赋值，指定实例绑定的安全组。
- **mode**：通过引用输入变量 instance_mode 进行赋值，指定实例类型。
- **datastore.type**：通过引用输入变量 database_type 进行赋值，指定数据库类型。
- **datastore.version**：通过引用输入变量 database_version 进行赋值，指定数据库版本。
- **datastore.storage_engine**：通过引用输入变量 storage_engine 进行赋值，指定存储引擎。
- **flavor.type**：通过引用输入变量 node_type 进行赋值，指定节点类型。
- **flavor.num**：通过引用输入变量 node_number 进行赋值，指定节点数量。
- **flavor.spec_code**：通过引用输入变量 node_spec_code 进行赋值，指定节点规格。
- **flavor.storage**：通过引用输入变量 node_storage_type 进行赋值，指定存储类型。
- **flavor.size**：通过引用输入变量 node_size 进行赋值，指定磁盘大小。
- **flavor.node_list**：通过引用输入变量 node_list 进行赋值，指定待删除的节点ID列表。

### 7. 查询DDS实例信息

在TF文件（如main.tf）中添加以下脚本：

```hcl
# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下查询DDS实例信息
data "huaweicloud_dds_instances" "test" {
  name = huaweicloud_dds_instance.test.name

  depends_on = [huaweicloud_dds_instance.test]
}

locals {
  nodeId = try([for v in flatten(data.huaweicloud_dds_instances.test.instances[*].groups[*].nodes) : v if v.role == "Primary"][0].id, "")
}
```

**参数说明**：
- **name**：通过引用 huaweicloud_dds_instance.test.name 进行赋值，指定要查询的DDS实例名称。
- **depends_on**：显式依赖 huaweicloud_dds_instance.test，确保实例创建完成后再查询。
- **locals.nodeId**：从查询结果中提取主节点的ID，用于后续绑定EIP。

### 8. 绑定弹性公网IP到DDS实例

在TF文件（如main.tf）中添加以下脚本：

```hcl
# 在指定region（region参数缺省时默认继承当前provider块中所指定的region）下将弹性公网IP绑定到DDS实例
resource "huaweicloud_dds_instance_eip_associate" "test" {
  instance_id = huaweicloud_dds_instance.test.id
  node_id     = local.nodeId
  public_ip   = huaweicloud_vpc_eip.test.address
}
```

**参数说明**：
- **instance_id**：通过引用 huaweicloud_dds_instance.test.id 进行赋值，指定要绑定的DDS实例ID。
- **node_id**：通过引用 local.nodeId 进行赋值，指定要绑定的节点ID。
- **public_ip**：通过引用 huaweicloud_vpc_eip.test.address 进行赋值，指定要绑定的弹性公网IP地址。

### 9. 预设资源部署所需的入参（可选）

本实践中，部分资源、数据源使用了输入变量对配置内容进行赋值，这些输入参数在后续部署时需要手工输入。
同时，Terraform提供了通过`tfvars`文件预设这些配置的方法，可以避免每次执行时重复输入。

在工作目录下创建`terraform.tfvars`文件，示例内容如下：

```hcl
# 根据脚本变量填写；敏感信息使用占位符
vpc_name            = "your_vpc_name"
subnet_name         = "your_subnet_name"
security_group_name = "your_security_group_name"
eip_bandwidth_name  = "your_eip_bandwidth_name"
instance_name       = "your_instance_name"
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

### 10. 初始化并应用Terraform配置

完成以上脚本配置后，执行以下步骤来创建资源：

1. 运行 `terraform init` 初始化环境
2. 运行 `terraform plan` 查看资源创建计划
3. 确认资源计划无误后，运行 `terraform apply` 开始创建DDS实例并绑定弹性公网IP
4. 运行 `terraform show` 查看已创建的DDS实例及弹性公网IP绑定信息

## 参考信息

- [华为云文档数据库服务产品文档](https://support.huaweicloud.com/dds/index.html)
- [华为云Provider文档](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs)
- [DDS实例绑定弹性公网IP最佳实践源码参考](https://github.com/huaweicloud/terraform-provider-huaweicloud/tree/master/examples/dds/dds-associate-eip)
