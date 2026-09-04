# Deploy Redis Instance All Sessions Kill

## Application Scenario

Distributed Cache Service (DCS) is a high-performance, highly available in-memory database service provided by Huawei Cloud, supporting mainstream cache engines such as Redis and Memcached. When a Redis instance has a large number of abnormal or idle client sessions, it may occupy connection resources and affect the stability and performance of the instance.

This best practice will introduce how to use Terraform to deploy a DCS Redis single-node instance on Huawei Cloud and use the `huaweicloud_dcs_all_sessions_kill` resource to kill all client sessions of the instance. This practice covers the automatic creation and configuration of VPC, subnet, availability zone, instance flavor, and other resources, helping you quickly master the automated deployment process of DCS instance session management.

## Related Resources/Data Sources

This best practice involves the following main resources and data sources:

### Data Sources

- [Availability Zones (data.huaweicloud_availability_zones)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/data-sources/availability_zones)
- [DCS Flavors (data.huaweicloud_dcs_flavors)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/data-sources/dcs_flavors)
- [DCS Instance Shards (data.huaweicloud_dcs_instance_shards)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/data-sources/dcs_instance_shards)

### Resources

- [Virtual Private Cloud (huaweicloud_vpc)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/vpc)
- [Subnet (huaweicloud_vpc_subnet)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/vpc_subnet)
- [Random Password (random_password)](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/password)
- [DCS Redis Instance (huaweicloud_dcs_instance)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/dcs_instance)
- [DCS All Sessions Kill (huaweicloud_dcs_all_sessions_kill)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/dcs_all_sessions_kill)

### Resource/Data Source Dependencies

```
huaweicloud_vpc
    └── huaweicloud_vpc_subnet
          └── huaweicloud_dcs_instance
                ├── data.huaweicloud_dcs_instance_shards
                └── huaweicloud_dcs_all_sessions_kill
```

## Operation Steps

### 1. Script Preparation

Prepare the TF file (such as main.tf) for writing the current best practice script in the specified workspace, and ensure that it (or other TF files in the same directory) contains the provider version declaration and Huawei Cloud authentication information required for deploying resources.
For configuration introduction, refer to [Preparation Before Deploying Huawei Cloud Resources](../../introductions/prepare_before_deploy.md).

### 2. Create VPC

Add the following script to the TF file (such as main.tf):

```hcl
# Create VPC in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
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

**Parameter Description**:
- **name**: Assigned by referencing the input variable vpc_name, used to specify the name of the VPC.
- **cidr**: Assigned by referencing the input variable vpc_cidr, used to specify the CIDR block of the VPC.

### 3. Create Subnet

Add the following script to the TF file (such as main.tf):

```hcl
# Create subnet in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
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

**Parameter Description**:
- **vpc_id**: Assigned by referencing huaweicloud_vpc.test.id, specifying the VPC to which the subnet belongs.
- **name**: Assigned by referencing the input variable subnet_name, used to specify the name of the subnet.
- **cidr**: Assigned by referencing the input variable subnet_cidr, if not set, it is automatically divided from the VPC CIDR.
- **gateway_ip**: Assigned by referencing the input variable subnet_gateway_ip, if not set, the gateway address is automatically calculated.

### 4. Query Availability Zones

Add the following script to the TF file (such as main.tf):

```hcl
# Query availability zones in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
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

**Parameter Description**:
- **count**: When no availability zone is specified, the data source is created through a conditional expression to query the available availability zone list.

### 5. Query DCS Flavors

Add the following script to the TF file (such as main.tf):

```hcl
# Query DCS flavors in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
data "huaweicloud_dcs_flavors" "test" {
  count = var.instance_flavor_id == "" ? 1 : 0

  engine         = "Redis"
  capacity       = var.instance_capacity
  engine_version = var.instance_engine_version
}
```

**Parameter Description**:
- **count**: When no instance flavor ID is specified, the data source is created through a conditional expression to query available DCS flavors.
- **engine**: Specifies the cache engine as Redis.
- **capacity**: Assigned by referencing the input variable instance_capacity, specifying the instance capacity.
- **engine_version**: Assigned by referencing the input variable instance_engine_version, specifying the engine version.

### 6. Generate Random Password

Add the following script to the TF file (such as main.tf):

```hcl
# Generate random password in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
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

**Parameter Description**:
- **count**: When no instance password is specified, the random password resource is created through a conditional expression.
- **length**: Specifies the length of the generated password as 12.
- **special**: Specifies that the generated password contains special characters.
- **override_special**: Specifies the allowed set of special characters.
- **min_upper**: Specifies at least 1 uppercase letter.
- **min_lower**: Specifies at least 1 lowercase letter.
- **min_numeric**: Specifies at least 1 digit.
- **min_special**: Specifies at least 1 special character.

### 7. Create DCS Redis Instance

Add the following script to the TF file (such as main.tf):

```hcl
# Create DCS Redis instance in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
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

**Parameter Description**:
- **name**: Assigned by referencing the input variable instance_name, used to specify the name of the DCS instance.
- **engine**: Specifies the cache engine as Redis.
- **engine_version**: Assigned by referencing the input variable instance_engine_version, specifying the engine version.
- **capacity**: Assigned by referencing the input variable instance_capacity, specifying the instance capacity.
- **vpc_id**: Assigned by referencing huaweicloud_vpc.test.id, specifying the VPC to which the instance belongs.
- **subnet_id**: Assigned by referencing huaweicloud_vpc_subnet.test.id, specifying the subnet to which the instance belongs.
- **password**: Assigned by referencing the input variable instance_password, if not set, the randomly generated password is used.
- **flavor**: Assigned by referencing the input variable instance_flavor_id, if not set, the first queried flavor is used.
- **availability_zones**: Assigned by referencing the input variable availability_zone, if not set, the first queried availability zone is used.

### 8. Query DCS Instance Shards

Add the following script to the TF file (such as main.tf):

```hcl
# Query DCS instance shards in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
data "huaweicloud_dcs_instance_shards" "test" {
  instance_id = huaweicloud_dcs_instance.test.id
}
```

**Parameter Description**:
- **instance_id**: Assigned by referencing huaweicloud_dcs_instance.test.id, specifying the DCS instance whose shard information is to be queried.

### 9. Kill All Client Sessions of DCS Instance

Add the following script to the TF file (such as main.tf):

```hcl
# Kill all client sessions of DCS instance in the specified region (if the region parameter is omitted, it inherits the region specified in the current provider block)
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

**Parameter Description**:
- **instance_id**: Assigned by referencing huaweicloud_dcs_instance.test.id, specifying the DCS instance whose sessions are to be killed.
- **kill_all_nodes**: Assigned by referencing the input variable kill_all_nodes, specifying whether to kill sessions of all nodes.
- **node_id**: Assigned by referencing the node ID in local.replication_list, used to specify the node when kill_all_nodes is false.

### 10. Preset Input Parameters Required for Resource Deployment (Optional)

In this practice, some resources and data sources use input variables to assign values to configurations. These input parameters need to be manually entered during subsequent deployment.
Meanwhile, Terraform provides a method to preset these configurations through `tfvars` files, which can avoid repeated input during each execution.

Create a `terraform.tfvars` file in the working directory. The example content is as follows:

```hcl
# Fill in according to the script variables; use placeholders for sensitive information
vpc_name      = "tf_test_dcs_instance_vpc"
subnet_name   = "tf_test_dcs_instance_subnet"
instance_name = "tf_test_dcs_instance"
```

**Usage**:

1. Save the above content as the `terraform.tfvars` file in the working directory (this file name allows Terraform to automatically import the content of the `tfvars` file when executing terraform commands; other names need to add `.auto` before tfvars, such as `variables.auto.tfvars`)
2. Modify the parameter values according to actual needs
3. When executing `terraform plan` or `terraform apply`, Terraform will automatically read the variable values from this file

In addition to using the `terraform.tfvars` file, you can also set variable values in the following ways:

1. Command line parameters: `terraform apply -var="vpc_name=my-vpc"`
2. Environment variables: `export TF_VAR_vpc_name=my-vpc`
3. Custom-named variable files: `terraform apply -var-file="custom.tfvars"`

> Note: If the same variable is set in multiple ways, Terraform will use the variable value according to the following priority: command line parameters > variable files > environment variables > default values.

### 11. Initialize and Apply Terraform Configuration

After completing the above script configuration, execute the following steps to create resources:

1. Run `terraform init` to initialize the environment
2. Run `terraform plan` to view the resource creation plan
3. After confirming the resource plan is correct, run `terraform apply` to start creating the DCS Redis instance and killing its client sessions
4. Run `terraform show` to view the created DCS Redis instance and the session kill result

## Reference Information

- [Huawei Cloud DCS Product Documentation](https://support.huaweicloud.com/dcs/index.html)
- [Huawei Cloud Provider Documentation](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs)
- [Best Practice Source Code Reference For DCS Redis Instance All Sessions Kill](https://github.com/huaweicloud/terraform-provider-huaweicloud/tree/master/examples/dcs/redis-all-sessions-kill)
