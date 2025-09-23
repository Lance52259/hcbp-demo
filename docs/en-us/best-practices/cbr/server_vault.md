# Deploy Server Type Vault

## Application Scenario

Cloud Backup and Recovery (CBR) is a data protection service provided by Huawei Cloud, offering simple and easy-to-use backup services for both cloud and on-premises resources. When events such as virus intrusion, accidental deletion, or hardware/software failures occur, data can be restored to any backup point. Server type vault is a type of vault in CBR service, specifically designed for backing up Elastic Cloud Server (ECS) instances.

Server type vault supports complete backup of ECS instances, including system disks and data disks, ensuring that the entire server environment can be quickly restored when failures occur. This best practice will introduce how to use Terraform to automatically deploy a CBR server type vault, including creating ECS instances, configuring backup policies, and creating vaults.

## Related Resources/Data Sources

This best practice involves the following main resources and data sources:

### Data Sources

- [Availability Zones Query Data Source (data.huaweicloud_availability_zones)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/data-sources/availability_zones)
- [ECS Flavor List Query Data Source (data.huaweicloud_compute_flavors)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/data-sources/compute_flavors)
- [IMS Image List Query Data Source (data.huaweicloud_images_images)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/data-sources/images_images)

### Resources

- [VPC Resource (huaweicloud_vpc)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/vpc)
- [VPC Subnet Resource (huaweicloud_vpc_subnet)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/vpc_subnet)
- [Security Group Resource (huaweicloud_networking_secgroup)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/networking_secgroup)
- [ECS Instance Resource (huaweicloud_compute_instance)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/compute_instance)
- [CBR Backup Policy Resource (huaweicloud_cbr_policy)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/cbr_policy)
- [CBR Vault Resource (huaweicloud_cbr_vault)](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs/resources/cbr_vault)

### Resource/Data Source Dependencies

```
data.huaweicloud_availability_zones.test
    └── huaweicloud_vpc_subnet.test
        └── huaweicloud_compute_instance.test
            └── huaweicloud_cbr_vault.test

data.huaweicloud_compute_flavors.test
    └── huaweicloud_compute_instance.test

data.huaweicloud_images_images.test
    └── huaweicloud_compute_instance.test

huaweicloud_vpc.test
    └── huaweicloud_vpc_subnet.test

huaweicloud_networking_secgroup.test
    └── huaweicloud_compute_instance.test

huaweicloud_cbr_policy.test
    └── huaweicloud_cbr_vault.test
```

## Operation Steps

### 1. Script Preparation

Prepare the TF file (e.g., main.tf) in the specified workspace for writing the current best practice script, ensuring that it (or other TF files in the same directory) contains the provider version declaration and Huawei Cloud authentication information required for deploying resources.
Refer to the "Preparation Before Deploying Huawei Cloud Resources" document for configuration introduction.

### 2. Query Availability Zones Required for ECS Instance Resource Creation via Data Source

Add the following script to the TF file (e.g., main.tf) to instruct Terraform to perform a data source query, the result of which will be used to create the ECS instance:

```hcl
variable "availability_zone" {
  description = "Availability zone information for the ECS instance"
  type        = string
  default     = ""
}

# Get all availability zone information under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block), used to create the ECS instance
data "huaweicloud_availability_zones" "test" {}
```

**Parameter Description**:
- No special parameters, gets all availability zone information in the current region

### 3. Query ECS Instance Flavor Information via Data Source

Add the following script to the TF file (e.g., main.tf) to instruct Terraform to perform a data source query, the result of which will be used to create the ECS instance:

```hcl
variable "instance_flavor_id" {
  description = "ECS instance flavor ID, if not specified, will use the first available flavor that meets the criteria"
  type        = string
  default     = ""
}

variable "instance_flavor_performance_type" {
  description = "ECS instance flavor performance type, used to query available flavors when instance_flavor_id is not specified"
  type        = string
  default     = "normal"
}

variable "instance_flavor_cpu_core_count" {
  description = "ECS instance flavor CPU core count, used to query available flavors when instance_flavor_id is not specified"
  type        = number
  default     = 2
}

variable "instance_flavor_memory_size" {
  description = "ECS instance flavor memory size (GB), used to query available flavors when instance_flavor_id is not specified"
  type        = number
  default     = 4
}

# Get all ECS flavor information that meets the criteria under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block), used to create the ECS instance
data "huaweicloud_compute_flavors" "test" {
  count = var.instance_flavor_id == "" ? 1 : 0

  availability_zone = var.availability_zone == "" ? try(data.huaweicloud_availability_zones.test.names[0], null) : var.availability_zone
  performance_type  = var.instance_flavor_performance_type
  cpu_core_count    = var.instance_flavor_cpu_core_count
  memory_size       = var.instance_flavor_memory_size
}
```

**Parameter Description**:
- **count**: Number of data source instances, used to control whether to execute the ECS flavor list query data source, only creates the data source when `var.instance_flavor_id` is empty
- **availability_zone**: Availability zone, prioritizes input variable, uses the first result from availability zone list query data source if empty
- **performance_type**: Performance type, used to filter ECS flavors
- **cpu_core_count**: CPU core count, used to filter ECS flavors
- **memory_size**: Memory size, used to filter ECS flavors

### 4. Query ECS Instance Image Information via Data Source

Add the following script to the TF file (e.g., main.tf) to instruct Terraform to perform a data source query, the result of which will be used to create the ECS instance:

```hcl
variable "instance_image_id" {
  description = "ECS instance image ID, if not specified, will use the first available image that meets the criteria"
  type        = string
  default     = ""
}

variable "instance_image_os_type" {
  description = "ECS instance image operating system type, used to query available images when instance_image_id is not specified"
  type        = string
  default     = "Ubuntu"
}

variable "instance_image_visibility" {
  description = "ECS instance image visibility, used to query available images when instance_image_id is not specified"
  type        = string
  default     = "public"
}

# Get all image information that meets the criteria under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block), used to create the ECS instance
data "huaweicloud_images_images" "test" {
  count = var.instance_image_id == "" ? 1 : 0

  flavor_id  = var.instance_flavor_id == "" ? try(data.huaweicloud_compute_flavors.test[0].ids[0], "") : var.instance_flavor_id
  os         = var.instance_image_os_type
  visibility = var.instance_image_visibility
}
```

**Parameter Description**:
- **count**: Number of data source instances, used to control whether to execute the image list query data source, only creates the data source when `var.instance_image_id` is empty
- **flavor_id**: Flavor ID, prioritizes input variable, uses the first result from ECS flavor list query data source if empty
- **os**: Operating system type, used to filter images
- **visibility**: Visibility, used to filter images

### 5. Create VPC

Add the following script to the TF file (e.g., main.tf) to instruct Terraform to create a VPC resource:

```hcl
variable "vpc_name" {
  description = "VPC name"
  type        = string
}

variable "vpc_cidr" {
  description = "VPC CIDR block"
  type        = string
  default     = "192.168.0.0/16"
}

# Create a VPC resource under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block)
resource "huaweicloud_vpc" "test" {
  name = var.vpc_name
  cidr = var.vpc_cidr
}
```

**Parameter Description**:
- **name**: VPC name, assigned by referencing the input variable vpc_name
- **cidr**: VPC CIDR block, assigned by referencing the input variable vpc_cidr

### 6. Create VPC Subnet

Add the following script to the TF file to instruct Terraform to create a VPC subnet resource:

```hcl
variable "subnet_name" {
  description = "Subnet name"
  type        = string
}

variable "subnet_cidr" {
  description = "Subnet CIDR block, if not specified, will calculate a subnet CIDR within the existing CIDR address block"
  type        = string
  default     = ""
}

variable "subnet_gateway_ip" {
  description = "Subnet gateway IP, if not specified, will calculate a gateway IP within the existing CIDR address block"
  type        = string
  default     = ""
}

# Create a VPC subnet resource under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block)
resource "huaweicloud_vpc_subnet" "test" {
  vpc_id            = huaweicloud_vpc.test.id
  name              = var.subnet_name
  cidr              = var.subnet_cidr == "" ? cidrsubnet(huaweicloud_vpc.test.cidr, 8, 0) : var.subnet_cidr
  gateway_ip        = var.subnet_gateway_ip == "" ? cidrhost(cidrsubnet(huaweicloud_vpc.test.cidr, 8, 0), 1) : var.subnet_gateway_ip
  availability_zone = var.availability_zone == "" ? try(data.huaweicloud_availability_zones.test.names[0], null) : var.availability_zone
}
```

**Parameter Description**:
- **vpc_id**: VPC ID, assigned by referencing the VPC resource (huaweicloud_vpc.test) ID
- **name**: Subnet name, assigned by referencing the input variable subnet_name
- **cidr**: Subnet CIDR block, prioritizes input variable, calculates using cidrsubnet function if empty
- **gateway_ip**: Gateway IP, prioritizes input variable, calculates using cidrhost function if empty
- **availability_zone**: Availability zone, prioritizes input variable, uses the first result from availability zone list query data source if empty

### 7. Create Security Group

Add the following script to the TF file to instruct Terraform to create a security group resource:

```hcl
variable "secgroup_name" {
  description = "Security group name"
  type        = string
}

# Create a security group resource under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block)
resource "huaweicloud_networking_secgroup" "test" {
  name                 = var.secgroup_name
  delete_default_rules = true
}
```

**Parameter Description**:
- **name**: Security group name, assigned by referencing the input variable secgroup_name
- **delete_default_rules**: Whether to delete default rules, set to true to delete default security group rules

### 8. Create ECS Instance

Add the following script to the TF file to instruct Terraform to create an ECS instance resource:

```hcl
variable "ecs_instance_name" {
  description = "ECS instance name"
  type        = string
}

variable "key_pair_name" {
  description = "ECS login key pair name"
  type        = string
  default     = ""
}

variable "system_disk_type" {
  description = "System disk type"
  type        = string
  default     = "SAS"
}

variable "system_disk_size" {
  description = "System disk size (GB)"
  type        = number
  default     = 40
}

# Create an ECS instance resource under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block)
resource "huaweicloud_compute_instance" "test" {
  name              = var.ecs_instance_name
  availability_zone = var.availability_zone == "" ? try(data.huaweicloud_availability_zones.test.names[0], null) : var.availability_zone
  flavor_id         = var.instance_flavor_id == "" ? try(data.huaweicloud_compute_flavors.test[0].flavors[0].id, "") : var.instance_flavor_id
  image_id          = var.instance_image_id == "" ? try(data.huaweicloud_images_images.test[0].images[0].id, "") : var.instance_image_id
  security_groups   = [huaweicloud_networking_secgroup.test.name]
  key_pair          = var.key_pair_name
  system_disk_type  = var.system_disk_type
  system_disk_size  = var.system_disk_size

  network {
    uuid = huaweicloud_vpc_subnet.test.id
  }
}
```

**Parameter Description**:
- **name**: ECS instance name, assigned by referencing the input variable ecs_instance_name
- **availability_zone**: Availability zone, prioritizes input variable, uses the first result from availability zone list query data source if empty
- **flavor_id**: Flavor ID, prioritizes input variable, uses the first result from ECS flavor list query data source if empty
- **image_id**: Image ID, prioritizes input variable, uses the first result from image list query data source if empty
- **security_groups**: Security group list, assigned by referencing the security group resource (huaweicloud_networking_secgroup.test) name
- **key_pair**: Key pair name, assigned by referencing the input variable key_pair_name
- **system_disk_type**: System disk type, assigned by referencing the input variable system_disk_type
- **system_disk_size**: System disk size, assigned by referencing the input variable system_disk_size
- **network.uuid**: Network ID, assigned by referencing the VPC subnet resource (huaweicloud_vpc_subnet.test) ID

### 9. Create CBR Backup Policy (Optional)

Add the following script to the TF file to instruct Terraform to create a CBR backup policy resource:

```hcl
variable "enable_policy" {
  description = "Whether to enable backup policy"
  type        = bool
  default     = false
}

# Create a CBR backup policy resource under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block)
resource "huaweicloud_cbr_policy" "test" {
  count = var.enable_policy ? 1 : 0

  name        = "${var.vault_name}-policy"
  type        = "backup"
  time_period = 20
  time_zone   = "UTC+08:00"
  enabled     = true

  backup_cycle {
    days            = "MO,TU"
    execution_times = ["06:00"]
  }
}
```

**Parameter Description**:
- **count**: Number of resource instances, used to control whether to create the backup policy resource, only creates when `var.enable_policy` is true
- **name**: Backup policy name, assigned by referencing the input variable vault_name and fixed suffix
- **type**: Policy type, set to "backup" for backup policy
- **time_period**: Backup retention time (days), set to 20 days
- **time_zone**: Time zone, set to "UTC+08:00"
- **enabled**: Whether to enable policy, set to true
- **backup_cycle.days**: Backup cycle, set to Monday and Tuesday
- **backup_cycle.execution_times**: Execution time, set to 06:00

### 10. Create CBR Vault

Add the following script to the TF file to instruct Terraform to create a CBR vault resource:

```hcl
variable "vault_name" {
  description = "CBR vault name"
  type        = string
}

variable "protection_type" {
  description = "Vault protection type (backup or replication)"
  type        = string
  default     = "backup"

  validation {
    condition     = contains(["backup", "replication"], var.protection_type)
    error_message = "Protection type must be 'backup' or 'replication'."
  }
}

variable "consistent_level" {
  description = "Vault consistency level (crash_consistent or app_consistent)"
  type        = string
  default     = "crash_consistent"

  validation {
    condition     = contains(["crash_consistent", "app_consistent"], var.consistent_level)
    error_message = "Consistency level must be 'crash_consistent' or 'app_consistent'."
  }
}

variable "vault_size" {
  description = "CBR vault size (GB)"
  type        = number
}

variable "auto_bind" {
  description = "Whether to automatically bind vault to policy"
  type        = bool
  default     = false
}

variable "auto_expand" {
  description = "Whether to automatically expand when vault is full"
  type        = bool
  default     = false
}

variable "enterprise_project_id" {
  description = "Enterprise project ID to which the vault belongs"
  type        = string
  default     = "0"
}

variable "backup_name_prefix" {
  description = "Backup name prefix"
  type        = string
  default     = ""
}

variable "is_multi_az" {
  description = "Whether the vault is deployed across AZs"
  type        = bool
  default     = false
}

variable "exclude_volumes" {
  description = "Whether to exclude volume backup"
  type        = bool
  default     = false
}

variable "tags" {
  description = "Vault tags"
  type        = map(string)
  default     = {
    environment = "test"
    terraform   = "true"
  }
}

# Create a CBR vault resource under the specified region (if the region parameter is omitted, it defaults to the region specified in the current provider block)
resource "huaweicloud_cbr_vault" "test" {
  name                  = var.vault_name
  type                  = "server"
  protection_type       = var.protection_type
  consistent_level      = var.consistent_level
  size                  = var.vault_size
  auto_bind             = var.auto_bind
  auto_expand           = var.auto_expand
  enterprise_project_id = var.enterprise_project_id
  backup_name_prefix    = var.backup_name_prefix
  is_multi_az           = var.is_multi_az

  resources {
    server_id = huaweicloud_compute_instance.test.id
    excludes  = var.exclude_volumes ? [huaweicloud_compute_instance.test.system_disk_id] : []
  }

  dynamic "policy" {
    for_each = var.enable_policy ? [1] : []

    content {
      id = huaweicloud_cbr_policy.test[0].id
    }
  }

  tags = var.tags
}
```

**Parameter Description**:
- **name**: Vault name, assigned by referencing the input variable vault_name
- **type**: Vault type, set to "server" for server type vault
- **protection_type**: Protection type, assigned by referencing the input variable protection_type
- **consistent_level**: Consistency level, assigned by referencing the input variable consistent_level
- **size**: Vault size, assigned by referencing the input variable vault_size
- **auto_bind**: Whether to auto-bind, assigned by referencing the input variable auto_bind
- **auto_expand**: Whether to auto-expand, assigned by referencing the input variable auto_expand
- **enterprise_project_id**: Enterprise project ID, assigned by referencing the input variable enterprise_project_id
- **backup_name_prefix**: Backup name prefix, assigned by referencing the input variable backup_name_prefix
- **is_multi_az**: Whether to deploy across AZs, assigned by referencing the input variable is_multi_az
- **resources.server_id**: Server ID, assigned by referencing the ECS instance resource (huaweicloud_compute_instance.test) ID
- **resources.excludes**: Excluded volumes, determined by exclude_volumes variable whether to exclude system disk
- **policy.id**: Policy ID, assigned by referencing the backup policy resource (huaweicloud_cbr_policy.test) ID when policy is enabled
- **tags**: Tags, assigned by referencing the input variable tags

### 11. Preset Input Parameters Required for Resource Deployment (Optional)

In this practice, some resources and data sources use input variables to assign values to configuration content. These input parameters need to be manually entered during subsequent deployments.
At the same time, Terraform provides a method to preset these configurations through `.tfvars` files, which can avoid repeated input during each execution.

Create a `terraform.tfvars` file in the working directory with the following example content:

```hcl
# Authentication information
region_name = "cn-north-4"
access_key  = "your-access-key"
secret_key  = "your-secret-key"

# Network configuration
vpc_name    = "cbr-test-vpc"
subnet_name = "cbr-test-subnet"
secgroup_name = "cbr-test-sg"

# ECS instance configuration
ecs_instance_name = "cbr-test-ecs"
key_pair_name    = "your-key-pair"

# CBR vault configuration
vault_name        = "cbr-vault-server"
vault_size        = 200
enable_policy     = true
protection_type   = "backup"
consistent_level  = "crash_consistent"
auto_bind         = false
auto_expand       = false
exclude_volumes   = false

# Tag configuration
tags = {
  environment = "test"
  project     = "cbr-demo"
  terraform   = "true"
}
```

**Usage**:

1. Save the above content as `terraform.tfvars` file in the working directory (this file name allows users to automatically import the content of this `tfvars` file when executing terraform commands; for other names, `.auto` needs to be added before tfvars, such as `variables.auto.tfvars`)
2. Modify parameter values as needed
3. When executing `terraform plan` or `terraform apply`, Terraform will automatically read the variable values from this file

In addition to using `terraform.tfvars` file, variable values can also be set in the following ways:

1. Command line parameters: `terraform apply -var="vpc_name=my-vpc" -var="subnet_name=my-subnet"`
2. Environment variables: `export TF_VAR_vpc_name=my-vpc`
3. Custom named variable files: `terraform apply -var-file="custom.tfvars"`

> Note: If the same variable is set in multiple ways, Terraform will use the variable value according to the following priority: command line parameters > variable files > environment variables > default values.

### 12. Initialize and Apply Terraform Configuration

After completing the above script configuration, execute the following steps to create resources:

1. Run `terraform init` to initialize the environment
2. Run `terraform plan` to view the resource creation plan
3. After confirming the resource plan is correct, run `terraform apply` to start creating the CBR server type vault
4. Run `terraform show` to view the details of the created CBR server type vault

## Reference Information

- [Huawei Cloud CBR Product Documentation](https://support.huaweicloud.com/cbr/index.html)
- [Huawei Cloud Provider Documentation](https://registry.terraform.io/providers/huaweicloud/huaweicloud/latest/docs)
- [CBR Best Practice Source Code Reference](https://github.com/huaweicloud/terraform-provider-huaweicloud/tree/master/examples/cbr)
